// -------------------------
// Robust filter-init + URL sync (no triggering of click handlers)
// Waits for window.onload and re-applies final URL to survive other scripts.
// -------------------------

// Keep normal click handler for user interactions (unchanged)
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

    window.history.pushState({ path: newUrl }, '', newUrl);
});

// helper - create tag bubble if not present
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

// build and set final URL (preserve other params). useReplace true => replaceState
function setCombinedFilterUrl(useReplace = true) {
  const original = new URL(window.location.href);
  const originalParams = new URLSearchParams(original.search);

  const finalParams = new URLSearchParams();
  // preserve all params except our three filter keys
  for (const [k, v] of originalParams.entries()) {
    if (k !== 'region' && k !== 'industry' && k !== 'role') {
      finalParams.append(k, v);
    }
  }

  // append filters from DOM
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
    console.log('[filters] replaceState set to:', newUrl);
  } else {
    window.history.pushState({ path: newUrl }, '', newUrl);
    console.log('[filters] pushState set to:', newUrl);
  }
}

// wait for checkboxes to exist (or timeout) then run callback
function waitForCheckboxes(cb, maxWaitMs = 7000, interval = 100) {
  const start = Date.now();
  const iv = setInterval(() => {
    if ($('.category-checkbox').length) {
      clearInterval(iv);
      cb();
    } else if (Date.now() - start > maxWaitMs) {
      clearInterval(iv);
      console.warn('[filters] waitForCheckboxes timed out, running callback anyway');
      cb();
    }
  }, interval);
}

// Activate checkboxes from the querystring WITHOUT triggering click handlers
function activateFiltersFromQuery() {
  const originalQuery = window.location.href.split('?')[1] || '';
  const params = new URLSearchParams(originalQuery);

  const regions = params.getAll("region").map(v => (v || '').toString().toLowerCase().trim());
  const industries = params.getAll("industry").map(v => (v || '').toString().toLowerCase().trim());
  const roles = params.getAll("role").map(v => (v || '').toString().toLowerCase().trim());

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
      if (!$cb.hasClass('active')) $cb.addClass('active');
      // check the input if present
      const $input = $cb.find('.checkbox__trigger');
      if ($input.length) $input.prop('checked', true);

      // create tag bubble for UI
      const tagValue = regionAttr || industryAttr || roleAttr;
      ensureTagBubble(tagValue, displayText, false);

      // make wrapper visible if previously hidden
      const $wrapper = $cb.closest('.checkbox-wrapper-33');
      if ($wrapper.length && $wrapper.is(':hidden')) {
        $wrapper.show();
      }
    }
  });

  // call your existing helpers safely (they depend on DOM state)
  if (typeof tagCheckRemaining === 'function') {
    try { tagCheckRemaining(); } catch (e) { console.warn('tagCheckRemaining error', e); }
  }
  if (typeof countCard === 'function') {
    try { countCard(); } catch (e) { console.warn('countCard error', e); }
  }
  if (typeof addOrRemoveClearFiltersLink === 'function') {
    try { addOrRemoveClearFiltersLink(); } catch (e) { console.warn('addOrRemoveClearFiltersLink error', e); }
  }

  // set final URL now (replaceState to avoid creating an init history entry)
  setTimeout(() => setCombinedFilterUrl(true), 150);
}


// MAIN: wait for full window load, then wait for checkboxes, then activate
// Using window.onload makes this run after other resources / scripts that run on load.
window.addEventListener('load', function () {
  // wait for the checkbox DOM to exist
  waitForCheckboxes(function () {
    // activate the filters from the querystring (without triggering clicks)
    activateFiltersFromQuery();

    // Re-apply final combined URL a couple more times to beat other late scripts that may rewrite it.
    // This is pragmatic: it wins race conditions where another script modifies the URL a little later.
    // 1st reapply: after 500ms
    setTimeout(() => setCombinedFilterUrl(true), 500);
    // 2nd reapply: after 1500ms
    setTimeout(() => setCombinedFilterUrl(true), 1500);
    // 3rd reapply (optional): after 3000ms — uncomment if you see very late scripts
    // setTimeout(() => setCombinedFilterUrl(true), 3000);
  }, 7000, 120);
});
