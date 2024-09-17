<!-- Include jQuery library before this script -->
<script src="https://code.jquery.com/jquery-3.6.0.min.js"></script>
<script>
$(document).ready(function() {
    // Set the default filter to "all"
    $('.product-filter[data-product-filter="all"]').addClass('active');

    function filterProducts(filter) {
        if (filter === "all") {
            // Show all product cards
            $('.product-suite-card').fadeIn();
        } else {
            // Filter product cards based on data-product-filter
            $('.product-suite-card').each(function() {
                var tags = $(this).data('product-suite-tags');
                if (tags === filter) {
                    $(this).fadeIn();
                } else {
                    $(this).fadeOut();
                }
            });
        }

        // Update results count
        var visibleCount = $('.product-suite-card:visible').length;
        $('.suite-card-count').text(visibleCount);
    }

    // Click event handler for product filters
    $('.product-filter').on('click', function() {
        var filter = $(this).data('product-filter');

        // Update active filter class
        $('.product-filter').removeClass('active');
        $(this).addClass('active');

        // Call filter function
        filterProducts(filter);
    });

    // Initialize results count
    filterProducts("all");
});
</script>
