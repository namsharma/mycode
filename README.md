// -------------------------
// FILTER CHECKBOX INIT + URL SYNC
// -------------------------

// small global flag to suppress history updates while we restore state
var suppressHistoryPush = false;

/* -------------------------------------------------------
   Click handler for checkboxes (UI behavior preserved).
   We guard the history push so init can suppress it.
   ------------------------------------------------------- */
$(document).on('click', '.category-checkbox', function () {

    const selectedRegions = [];
    const selectedIndustries = [];
    const selectedRoles = [];

    $('.category-checkbox.active').each(function () {
        const region = $(this).data('region');
        const industry = $(this).data('industry-category');
        const role = $(this).data('persona-type');

        if (region) selectedRegions.push(region);
        if (industry) selectedIndustries.push(industry);
        if (role) selectedRoles.push(role);
    });

    const url = new URL(window.location.href);
    const params = new URLSearchParams(url.search);

    // Remove existing filter keys and append current ones
    params.delete('region');
    selectedRegions.forEach(v => params.append('region', v));

    params.delete('industry');
    selectedIndustries.forEach(v => params.append('industry', v));

    params.delete('role');
    selectedRoles.forEach(v => params.append('role', v));

    const newQuery = params.toString();
    const newUrl = `${url.pathname}${newQuery ? `?${newQuery}` : ''}`;

    if (!suppressHistoryPush) {
      // Normal user-driven clicks update history as before
      window.history.pushState({ path: newUrl }, '', newUrl);
    } else {
      // suppressed during init to avoid partial/overwriting querystring
    }
});

/* -------------------------------------------------------
   Helper: build a combined URL from active checkboxes and
   preserve other existing params (like personalized, campaign).
   ------------------------------------------------------- */
function updateUrlFromActiveCheckboxes(useReplaceState = false) {
  const original = new URL(window.location.href);
  const originalParams = new URLSearchParams(original.search);

  // copy any params we should preserve (all except the three filter keys)
  const finalParams = new URLSearchParams();
  for (const [k, v] of originalParams.entries()) {
    if (k !== 'region' && k !== 'industry' && k !== 'role') {
      finalParams.append(k, v);
    }
  }

  // append the active filters (read from DOM)
  $('.category-checkbox.active').each(function () {
    const r = $(this).data('region');
    const ind = $(this).data('industry-category');
    const rl = $(this).data('persona-type');

    if (r) finalParams.append('region', r);
    if (ind) finalParams.append('industry', ind);
    if (rl) finalParams.append('role', rl);
  });

  const base = window.location.origin + window.location.pathname;
  const q = finalParams.toString();
  const newUrl = q ? base + '?' + q : base;

  if (useReplaceState) {
    window.history.replaceState({ path: newUrl }, '', newUrl);
  } else {
    window.history.pushState({ path: newUrl }, '', newUrl);
  }
}

/* -------------------------------------------------------
   Initialization: read query, trigger matching checkboxes,
   suppress per-click history updates and do one final push.
   - Normalizes case and trims whitespace when comparing.
   - Waits for .category-checkbox to exist before running.
   ------------------------------------------------------- */
$(document).ready(function () {
    const originalQuery = window.location.href.split('?')[1] || '';
    const params = new URLSearchParams(originalQuery);

    // Grab arrays of filter values from the querystring and normalize
    const regions = params.getAll("region").map(v => (v || '').toString().toLowerCase().trim());
    const industries = params.getAll("industry").map(v => (v || '').toString().toLowerCase().trim());
    const roles = params.getAll("role").map(v => (v || '').toString().toLowerCase().trim());

    // wait-for-element helper (polling). Runs callback once .category-checkbox exists or
    // times out after maxWaitMs.
    function waitForCheckboxesAndRun(callback, maxWaitMs = 3000, intervalMs = 100) {
      const start = Date.now();
      const iv = setInterval(function () {
        if ($('.category-checkbox').length) {
          clearInterval(iv);
          callback();
        } else if (Date.now() - start > maxWaitMs) {
          clearInterval(iv);
          // timed out — still try to run callback once to avoid leaving page unusable
          callback();
        }
      }, intervalMs);
    }

    function clickMatchingCheckboxes() {
        // Temporarily suppress per-click history updates
        suppressHistoryPush = true;

        $(".category-checkbox").each(function () {
            const $checkbox = $(this);

            // Read data attributes and normalize them for comparison
            const regionAttr = ($checkbox.data("region") || '').toString().toLowerCase().trim();
            const industryAttr = ($checkbox.data("industry-category") || '').toString().toLowerCase().trim();
            const roleAttr = ($checkbox.data("persona-type") || '').toString().toLowerCase().trim();

            const regionMatch = regionAttr && regions.length && regions.indexOf(regionAttr) > -1;
            const industryMatch = industryAttr && industries.length && industries.indexOf(industryAttr) > -1;
            const roleMatch = roleAttr && roles.length && roles.indexOf(roleAttr) > -1;

            if (regionMatch || industryMatch || roleMatch) {
                // Trigger the normal click handler so all UI side-effects run
                // (tag bubbles, counts, fadeIn/fadeOut etc.)
                // Ensure the element is visible first (if hidden by some parent) so click works reliably
                if (!$checkbox.is(':visible')) {
                  $checkbox.show();
                }
                $checkbox.trigger("click");
            }
        });

        // Give the click handlers time to finish DOM updates, then re-enable history push
        setTimeout(function () {
            suppressHistoryPush = false;
            // push a single combined URL reflecting all active filters
            // Use pushState so it behaves like manual clicks; change to replaceState(true) if you prefer no extra history entry.
            updateUrlFromActiveCheckboxes(false);
        }, 300); // adjust if your page requires more time
    }

    // Run when checkboxes present (or after timeout)
    waitForCheckboxesAndRun(clickMatchingCheckboxes, 5000, 100);
});
