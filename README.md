(function($){
  // Map data-* attribute -> query param name
  const ATTR_TO_PARAM = {
    'data-region': 'region',
    'data-industry': 'industry',
    'data-persona-type': 'role'
  };

  // Read current URLSearchParams into a plain object of arrays
  function readAllParams() {
    const usp = new URLSearchParams(window.location.search);
    const out = {};
    for (const key of usp.keys()) {
      out[key] = usp.getAll(key);
    }
    return out;
  }

  // Rebuild URL's search string from params object (object values are arrays)
  function writeParams(paramsObj) {
    const usp = new URLSearchParams();
    Object.keys(paramsObj).forEach(k => {
      (paramsObj[k] || []).forEach(v => {
        if (v !== undefined && v !== null && v !== '') usp.append(k, v);
      });
    });
    const newUrl = window.location.pathname + (usp.toString() ? '?' + usp.toString() : '') + window.location.hash;
    history.replaceState(null, '', newUrl);
  }

  // Add a value (avoid duplicate) to a param array
  function addParamValue(paramsObj, key, value) {
    paramsObj[key] = paramsObj[key] || [];
    if (!paramsObj[key].includes(value)) paramsObj[key].push(value);
  }

  // Remove a single value from param array (and delete key if empty)
  function removeParamValue(paramsObj, key, value) {
    if (!paramsObj[key]) return;
    paramsObj[key] = paramsObj[key].filter(v => v !== value);
    if (paramsObj[key].length === 0) delete paramsObj[key];
  }

  // Given a .category-checkbox element, detect which param it corresponds to and its value
  function getParamInfo($elem) {
    for (const attr in ATTR_TO_PARAM) {
      const val = $elem.attr(attr);
      if (typeof val !== 'undefined') {
        return { key: ATTR_TO_PARAM[attr], value: String(val) };
      }
    }
    return null;
  }

  // On change handler for inputs (delegated)
  $(document).on('change', '.checkbox__trigger', function(e){
    const $input = $(this);
    const $category = $input.closest('.category-checkbox');
    const info = getParamInfo($category);
    if (!info) return; // nothing to do if structure unexpected

    const params = readAllParams();
    if ($input.prop('checked')) {
      addParamValue(params, info.key, info.value);
    } else {
      removeParamValue(params, info.key, info.value);
    }
    writeParams(params);
  });

  // Check checkboxes that match current URL params.
  // `suppressUrlUpdate` true means we will NOT change the URL while checking (used on init / dynamic addition).
  function applyParamsToCheckboxes(suppressUrlUpdate = true) {
    const params = readAllParams();
    // For each category-checkbox on page, check if its value is present in params
    $('.category-checkbox').each(function(){
      const $cat = $(this);
      const info = getParamInfo($cat);
      if (!info) return;
      const $input = $cat.find('.checkbox__trigger').first();
      const shouldBeChecked = (params[info.key] || []).indexOf(info.value) !== -1;
      // Only change checked state if it's different
      if ($input.length) {
        if ($input.prop('checked') !== shouldBeChecked) {
          $input.prop('checked', shouldBeChecked);
          // If the site has other UI updates tied to clicking labels, trigger change event but avoid URL write
          if (suppressUrlUpdate) {
            $input.triggerHandler('change'); // triggerHandler doesn't bubble and doesn't call delegated handler; avoids URL update
          } else {
            $input.trigger('change'); // this will call handler and update URL
          }
        }
      }
    });
  }

  // Run on initial load
  $(function(){
    applyParamsToCheckboxes(true);
  });

  // Observe DOM for dynamically added .category-checkbox nodes (for Industry / Persona)
  // When found, applyParamsToCheckboxes for just those newly-added elements.
  (function watchForDynamicCheckboxes(){
    const observer = new MutationObserver(function(mutations){
      let found = false;
      mutations.forEach(m => {
        // check added nodes
        m.addedNodes && Array.from(m.addedNodes).forEach(node => {
          if (!(node instanceof HTMLElement)) return;
          // if the node itself is a category-checkbox or contains them, mark found
          if ($(node).is('.category-checkbox') || $(node).find('.category-checkbox').length) {
            found = true;
          }
        });
      });
      if (found) {
        // when dynamic elements appear, apply params to newly added ones (do not rewrite URL)
        applyParamsToCheckboxes(true);
      }
    });

    observer.observe(document.body, { childList: true, subtree: true });
  })();

  // Extra helper: allow link clicks that contain query params to set checkboxes automatically.
  // If you navigate via link (same page) with new params, the replaceState above won't trigger load.
  // So listen for popstate to handle back/forward or manual URL edits.
  window.addEventListener('popstate', function(){
    applyParamsToCheckboxes(true);
  });

})(jQuery);
