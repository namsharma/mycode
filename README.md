// -------------------------
// SAFER FILTER INIT + URL SYNC (no triggering of click handlers)
// -------------------------

// Keep existing click handler mostly unchanged (it still updates URL on user clicks)
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

    params.delete('region');
    selectedRegions.forEach(v => params.append('region', v));

    params.delete('industry');
    selectedIndustries.forEach(v => params.append('industry', v));

    params.delete('role');
    selectedRoles.forEach(v => params.append('role', v));

    const newQuery = params.toString();
    const newUrl = `${url.pathname}${newQuery ? `?${newQuery}` : ''}`;

    // user clicks should still push history
    window.history.pushState({ path: newUrl }, '', newUrl);
});


// Helper: create tag bubble if not already present
function ensureTagBubble(tagValue, displayText, isPrimary = false) {
  if (!tagValue) return;
  const sel = `.selected-tags-container .tag-bubble[data-tag-value="${tagValue}"]`;
  if ($(sel).length === 0) {
    const $bubble = $('<div class="tag-bubble" data-tag-value="' + tagValue + '">' +
      displayText +
      '<svg class="fis-icon fis-icon-x" aria-hidden="true"><use xlink:href="#fis-icon-x"></use></svg></div>');
    if (isPrimary) $bubble.addClass('primary-filter');
    $bubble.appendTo('.right-content-area .selected-tags-container .row');
  }
}


// Build final URL preserving unrelated params and set it (replaceState to avoid extra history entry)
function pushCombinedFiltersToUrl(useReplace = true) {
  const original = new URL(window.location.href);
  const originalParams = new URLSearchParams(original.search);

  const finalParams = new URLSearchParams();
  // preserve other params except region/industry/role
  for (const [k, v] of originalParams.entries()) {
    if (k !== 'region' && k !== 'industry' && k !== 'role') {
      finalParams.append(k, v);
    }
  }

  // append from DOM active checkboxes
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

  if (useReplace) {
    window.history.replaceState({ path: newUrl }, '', newUrl);
  } else {
    window.history.pushState({ path: newUrl }, '', newUrl);
  }
}


// Wait helper: call cb once .category-checkbox exists (or after timeout)
function waitForCheckboxes(cb, maxWaitMs = 5000, interval = 100) {
  const start = Date.now();
  const iv = setInterval(() => {
    if ($('.category-checkbox').length) {
      clearInterval(iv);
      cb();
    } else if (Date.now() - start > maxWaitMs) {
      clearInterval(iv);
      cb(); // try anyway
    }
  }, interval);
}


// Direct activation function: does NOT trigger clicks that may navigate
function activateFiltersFromQuery() {
  const originalQuery = window.location.href.split('?')[1] || '';
  const params = new URLSearchParams(originalQuery);

  // Normalize arrays (lowercase + trim)
  const regions = params.getAll("region").map(v => (v || '').toString().toLowerCase().trim());
  const industries = params.getAll("industry").map(v => (v || '').toString().toLowerCase().trim());
  const roles = params.getAll("role").map(v => (v || '').toString().toLowerCase().trim());

  // For each checkbox, if its data-* matches any param, set it active directly
  $(".category-checkbox").each(function () {
    const $cb = $(this);
    const regionAttr = ($cb.data("region") || '').toString().toLowerCase().trim();
    const industryAttr = ($cb.data("industry-category") || '').toString().toLowerCase().trim();
    const roleAttr = ($cb.data("persona-type") || '').toString().toLowerCase().trim();
    const displayText = $cb.text().trim();

    let shouldActivate = false;
    if (regionAttr && regions.length && regions.indexOf(regionAttr) > -1) shouldActivate = true;
    if (industryAttr && industries.length && industries.indexOf(industryAttr) > -1) shouldActivate = true;
    if (roleAttr && roles.length && roles.indexOf(roleAttr) > -1) shouldActivate = true;

    if (shouldActivate) {
      // mark active and check input
      if (!$cb.hasClass('active')) $cb.addClass('active');
      $cb.find('.checkbox__trigger').prop('checked', true);

      // add a tag bubble (avoid duplication)
      const tagValue = regionAttr || industryAttr || roleAttr;
      ensureTagBubble(tagValue, displayText, false);

      // if the checkbox wrapper was hidden by earlier hide logic, unhide it
      const $wrapper = $cb.closest('.checkbox-wrapper-33');
      if ($wrapper.length && $wrapper.is(':hidden')) {
        $wrapper.show();
      }
    }
  });

  // Now call the shared helpers that depend on the DOM state (do not trigger clicks)
  // Recalculate counts / show-more / etc. If these functions exist, call them.
  if (typeof tagCheckRemaining === 'function') {
    tagCheckRemaining();
  }
  if (typeof countCard === 'function') {
    countCard();
  }
  if (typeof addOrRemoveClearFiltersLink === 'function') {
    addOrRemoveClearFiltersLink();
  }

  // a short defer to allow show/hide animations to settle, then set final URL
  setTimeout(function () {
    // set final URL to reflect active filters; use replaceState to avoid polluting history
    pushCombinedFiltersToUrl(true);
  }, 150);
}


// Run initialization safely when ready
$(document).ready(function () {
  waitForCheckboxes(activateFiltersFromQuery, 5000, 100);
});
