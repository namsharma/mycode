<script>
$(document).ready(function() {
    // Function to check if the URL has the specified query parameter
    function hasQueryParam(param) {
        const urlParams = new URLSearchParams(window.location.search);
        return urlParams.has(param);
    }

    // Initialize the Slick slider if the query parameter "loc" is equal to "specs"
    if (hasQueryParam('loc') && new URLSearchParams(window.location.search).get('loc') === 'specs') {
        $('.your-slider-class').slick({
            // Add your Slick slider options here
            dots: true,
            infinite: true,
            speed: 300,
            slidesToShow: 1,
            slidesToScroll: 1
        });
    }
});
</script>
