// Helper function to check if a class exists on the page
function classExists(className) {
    return document.querySelector(className) !== null;
}

// Function to add the loading spinner
function showLoadingSpinner() {
    if (classExists(".search-results")) {
        $(".search-results").parent().prepend("<div class='searchloader'><span class='loaderimg'></span><p>Search results are loading...</p></div>");
    }
    if (classExists(".no-results")) {
        $(".no-results").css("display", "none");
    }
}

// Initial loading spinner and hide "no results" message
showLoadingSpinner();

// Set a timeout to add the spinner if the URL contains '#e'
setTimeout(function() {
    if (window.location.href.indexOf("#e") > 0) {
        showLoadingSpinner();
    }
}, 700);

// Click event for the search button
$(".search-box-button").click(function() {
    showLoadingSpinner();
});

// Event listener for "Enter" key in the search input field
const getinput = document.getElementById("searchbar__input_resultPage");
if (getinput) {
    getinput.addEventListener("keydown", function(n) {
        if (n.code === "Enter") {
            validate(n);
        }
    });
}

// MutationObserver to monitor changes in the search results
if (classExists(".search-results")) {
    const observer = new MutationObserver(function(mutations) {
        mutations.forEach(function(mutation) {
            if (mutation.type === "childList") {
                $(".searchloader").fadeOut();
            }
        });
    });

    observer.observe(document.querySelector(".search-results"), { childList: true, subtree: true });
}
