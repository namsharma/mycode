<script>
$(document).ready(function() {
    // Function to check if the URL has the specified query parameter
    function hasQueryParam(param) {
        const urlParams = new URLSearchParams(window.location.search);
        return urlParams.has(param);
    }

    // Function to initialize the Slick slider
    function initializeSlider() {
        $('.your-slider-class').slick({
            // Add your Slick slider options here
            dots: true,
            infinite: true,
            speed: 300,
            slidesToShow: 1,
            slidesToScroll: 1
        });
    }

    // Click event for the anchor tag
    $('#checkParam').on('click', function(event) {
        event.preventDefault(); // Prevent default link behavior
        if (hasQueryParam('loc') && new URLSearchParams(window.location.search).get('loc') === 'specs') {
            setTimeout(initializeSlider, 1000); // Delay initialization
        }
    });

    // Click event for the div
    $('#checkParamDiv').on('click', function() {
        if (hasQueryParam('loc') && new URLSearchParams(window.location.search).get('loc') === 'specs') {
            setTimeout(initializeSlider, 1000); // Delay initialization
        }
    });
});
</script>
