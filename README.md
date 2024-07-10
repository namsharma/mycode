
<style>
  .fis-facet-wrapper{display: flex;}
  .facet-item{margin-right: 20px;}
</style>

<script>

(document).ready(function() {

/* Clear filters code */

    $('.filter-item input[type="checkbox"]').on('change', function() {
        var labelContent = $(this).next('label').text().trim();
        var facetItemId = $(this).attr('id') + '-facet-item';

        if ($(this).is(':checked')) {
            // Check if facet item already exists
            if ($('#' + facetItemId).length === 0) {
                // Create and append facet item inside fis-facet-wrapper
                var facetItem = $('<div id="' + facetItemId + '" class="facet-item facet-item-filter"><span>' + labelContent + '</span><a href="#">X</a></div>');
                $('.fis-facet-wrapper').append(facetItem);

                // Event handler for 'X' click
                facetItem.find('a').on('click', function(e) {
                    e.preventDefault();
                    $('#' + facetItemId).remove();
                    // Uncheck the corresponding checkbox
                    $('#' + $(this).parent().attr('id').replace('-facet-item', '')).prop('checked', false).trigger('change');
                    // Check if 'Clear all filters' should be removed
                    checkClearAllFilters();
                });
            }

        } else {
            // If unchecked, remove corresponding facet item
            $('#' + facetItemId).remove();
            // Check if 'Clear all filters' should be removed
            checkClearAllFilters();
        }

        // Check if 'Clear all filters' should be added
        checkClearAllFilters();
    });

    // Function to add or remove 'Clear all filters' facet item
    function checkClearAllFilters() {
        var anyChecked = false;
        $('.filter-item input[type="checkbox"]').each(function() {
            if ($(this).is(':checked')) {
                anyChecked = true;
                return false; // Exit loop early if any checkbox is checked
            }
        });

        if (anyChecked) {
            // Ensure 'Clear all filters' is present
            if ($('.clear-all-filters').length === 0) {
                var clearAllItem = $('<div class="facet-item facet-item-filter clear-all-filters"><span>Clear all filters</span><a href="#">X</a></div>');
                
                // Prepend instead of append
                $('.fis-facet-wrapper').prepend(clearAllItem);

                // Event handler for 'X' click on 'Clear all filters'
                clearAllItem.find('a').on('click', function(e) {
                    e.preventDefault();
                    $('.facet-item-filter').remove();
                    $('.filter-item input[type="checkbox"]').prop('checked', false).trigger('change');
                });
            }
        } else {
            // Remove 'Clear all filters' if no checkboxes are checked
            $('.clear-all-filters').remove();
        }
    }

    /* Clear filters code */


    /* Get results count code */
    function updateResultsCount() {
        setTimeout(function(){
            var count = $('.fis-search-results__content:visible').length;
            $('.insight-result-count p').text("We found " + count + " results.");
        },500);
    }
    
    updateResultsCount();
    
    $('.filter-item input[type="checkbox"]').change(function() {
        updateResultsCount();
    });
    /* Get results count code */



   /* No results found code */
    checkResultsVisibility();

    // Listen for changes in checkboxes
    $('.filter-item input[type="checkbox"]').change(function() {
        // Delayed execution to ensure DOM update after checkbox change
        setTimeout(function() {
            // Check visibility again after checkbox change
            checkResultsVisibility();
        }, 500); // Adjust delay as necessary
    });

    // Function to check visibility of .fis-search-results__content and append/remove message
    function checkResultsVisibility() {
        if ($('.fis-search-results__content:visible').length === 0) {
            // If no such div is visible, append the no results found message
            setTimeout(function() {
            if ($('.no-results-found').length === 0) {
                $('.search-result-list').append('<li class="no-results-found">Sorry, No results are found.</li>');
            }
                }, 500);
        } else {
            // If there is at least one visible div with class .fis-search-results__content
            // Remove the no results found message if it exists
            $('.no-results-found').remove();
        }
    }
    /* No results found code */



    /* Explore more code */
    
    var $listItems = $('.search-result-list li');
  
    // Loop through all li elements starting from the 16th one
    $listItems.slice(15).css({
        'visibility': 'hidden',
        'opacity': '0',
        'height': '0'
    });

   
    $('.filter-item input[type="checkbox"]').change(function() {
        setTimeout(function() {
        var $alllistItems = $('.search-result-list li');    

        $alllistItems.css({
            'visibility': 'visible',
            'opacity': '1',
            'height': 'auto'
        });

        var $listItems = $('.search-result-list li:has(div.fis-search-results__content:visible)');
        var $visbleitems = $listItems.length;
        alert($visbleitems); 
    // Loop through all li elements starting from the 16th one
    $listItems.slice(15).css({
                'visibility': 'hidden',
                'opacity': '0',
                'height': '0'
            });
    }, 700);
    });


    $('.insght-srch-1dmr').click(function() {
        $('.search-result-list li').each(function() {
            $(this).css({
                'visibility': 'visible',
                'opacity': '1',
                'height': 'auto'
            });
        });
    $(this).hide(); // Hide the "Show All" button after clicking
  });

  /* Explore more code */

});
</script>
