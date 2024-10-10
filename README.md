
    (function($) {
        var checkboxes = $('.left-panel input[type="checkbox"]');
        var cards = $(".right-panel .card");

        checkboxes.on("change", function() {
            var selectedFilters = {
                dates: [],
                industries: [],
                topics: [],
                types: []
            };
            var filteredCards;

            // Gather selected filters
            checkboxes.filter(":checked").each(function() {
                var filterName = this.name.split('-')[1]; // Gets the part after the hyphen (e.g. 'date', 'insightind', etc.)
                
                if (filterName === "date") {
                    selectedFilters.dates.push(this.value);
                } else if (filterName === "insightind") {
                    selectedFilters.industries.push(this.value);
                } else if (filterName === "insighttop") {
                    selectedFilters.topics.push(this.value);
                } else if (filterName === "insighttyp") {
                    selectedFilters.types.push(this.value);
                }
            });

            filteredCards = cards;

            // Apply filtering for dates (cards will show if they match any of the selected dates)
            if (selectedFilters.dates.length > 0) {
                filteredCards = filteredCards.filter(function() {
                    var cardDates = $(this).data("date").toString().split("_");
                    var dateMatch = selectedFilters.dates.some(function(selectedDate) {
                        return cardDates.includes(selectedDate);
                    });
                    return dateMatch;
                });
            }

            // Apply other filters (industries, topics, etc.)
            if (selectedFilters.industries.length > 0) {
                filteredCards = filteredCards.filter(function() {
                    var cardIndustries = $(this).data("industry").split("_");
                    var industryMatch = selectedFilters.industries.some(function(selectedIndustry) {
                        return cardIndustries.includes(selectedIndustry);
                    });
                    return industryMatch;
                });
            }

            if (selectedFilters.topics.length > 0) {
                filteredCards = filteredCards.filter(function() {
                    var cardTopics = $(this).data("topic").split("_");
                    var topicMatch = selectedFilters.topics.some(function(selectedTopic) {
                        return cardTopics.includes(selectedTopic);
                    });
                    return topicMatch;
                });
            }

            if (selectedFilters.types.length > 0) {
                filteredCards = filteredCards.filter(function() {
                    var cardTypes = $(this).data("type").split("_");
                    var typeMatch = selectedFilters.types.some(function(selectedType) {
                        return cardTypes.includes(selectedType);
                    });
                    return typeMatch;
                });
            }

            // Hide all cards and then show the filtered ones
            cards.hide().filter(filteredCards).show();
        });

        // Initial display
        $(".right-panel .card").show();

        // Disable filters with zero counts
        $(".filter-item").each(function() {
            var filterCount = $(this).find("span.filter-counts").text();
            if (filterCount == 0) {
                $(this).addClass("disable-filter");
            }
        });
    })(jQuery);
