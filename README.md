(function ($) {
    function isValid($input) {
        var currentValue = $.trim($input.val());
        var lastValue = $input.data('lastValue') || "";
        return currentValue !== "" && currentValue !== lastValue;
    }

    function updateButtonState($input) {
        var $form = $input.closest('.searchbar__form');
        var $btn  = $form.find('.search-box-button');
        $btn.prop('disabled', !isValid($input));
    }

    $(document).on('focus', '.searchbar__form .search-box-input.searchbar__input.js-search-hero-input.tt-input', function () {
        var $input = $(this);
        var $form  = $input.closest('.searchbar__form');
        var $btn   = $form.find('.search-box-button');

        if (!$input.data('initialized')) {
            $btn.prop('disabled', true);       
            $input.data('lastValue', "");      
            $input.data('initialized', true); 
        }

        updateButtonState($input);
    });

    $(document).on('input keyup change', '.searchbar__form .search-box-input.searchbar__input.js-search-hero-input.tt-input', function () {
        var $input = $(this);
        updateButtonState($input);
    });

    $(document).on('submit', '.searchbar__form', function (e) {
        var $form  = $(this);
        var $input = $form.find('.search-box-input.searchbar__input.js-search-hero-input.tt-input').first();

        if (!isValid($input)) {
            e.preventDefault();
            e.stopImmediatePropagation();
            console.log('Search blocked: empty or same value');
            return false;
        }
        $input.data('lastValue', $.trim($input.val()));

        $form.find('.search-box-button').prop('disabled', true);
    });

    $(document).on('click', '.search-box-button', function (e) {
        var $btn   = $(this);
        var $form  = $btn.closest('.searchbar__form');
        var $input = $form.find('.search-box-input.searchbar__input.js-search-hero-input.tt-input').first();

        if (!isValid($input)) {
            e.preventDefault();
            e.stopImmediatePropagation();
            console.log('Button click blocked: empty or same value');
            return false;
        }
    });

})(jQuery);
