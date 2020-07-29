# Lab 4 - Scrape the web :page_facing_up:

In this lab you are going to write a function to scrape a web page. It is important to understand that this is different to web crawling. Web scraping will copy the HTML and you can then programmatically target individual elements, whereas web crawling will follow URLs until it reaches a page where no URLs are present.

### Step 1

1. Navigate to the `/pkg/actions/scaping.go` file
2. In this file, add the code below (read the code carefully, so you understand what is happening)

```golang

package actions

import (
	"encoding/json"
	"fmt"
	"github.com/gocolly/colly"
	"github.com/web-scraping/pkg/utils"
	"net/http"
	"os"
)

// Product struct will hold the information we want to print out from the web page
type Product struct {
	Name string
	Stars string
	Price string
}

/* Scrape is an exported function
** This lets functions outside of the "actions" package call it (makes it globally visible)
*/ 
func Scrape(w http.ResponseWriter, r *http.Request) {
	// Write the status code 200
	w.WriteHeader(http.StatusOK)

	// create a new collector with the colly HTTP framework
	c := colly.NewCollector()

	// Execute a new request on every call from the collector
	c.OnRequest(func(r *colly.Request) {
		fmt.Println("Visiting", r.URL)
	})

	// Create an empty slice based on the Product struct (Name, Stars, Price)
	var dataSlice []Product

	// Uses goquery to find an element of the HTML that we want to extract
	c.OnHTML("div.s-result-list.s-search-results.sg-row", func(e *colly.HTMLElement) {

		// There are many elements in the above HTML so use a ForEach to loop over them and then call the callback function
		e.ForEach("div.a-section.a-spacing-medium", func(_ int, e *colly.HTMLElement) {
			var productName, stars, price string

			// ChildText returns the stripped string from the within the matched element
			// In this case... the product name
			productName = e.ChildText("span.a-size-medium.a-color-base.a-text-normal")

			if productName == "" {
				// If we can't get any name, we return and go directly to the next element
				return
			}

			// In this case... the stars
			stars = e.ChildText("span.a-icon-alt")

			// Call the helper function to format the stars into a float (decimal) e.g 4.8
			format.FormatStars(&stars)

			// In this case... the price
			price = e.ChildText("span.a-price > span.a-offscreen")
			if price == "" {
				// If we can't get any price, we return and go directly to the next element
				return
			}

			// Format the price so it is readable - some prices may have a 'was' and a 'now' price
			// This helper function will strip the old price from the string
			format.FormatPrice(&price)

			//fmt.Printf("Product Name: %s \nStars: %s \nPrice: %s \n", productName, stars, price)

			// Append the collected data (Name, Stars, Price) from the element into the empty slice
			dataSlice = append(dataSlice, Product{
				Name: productName,
				Stars: stars,
				Price: price,
			})
		})

		// Marshal the JSON into a readable format using json.MarshalIndent
		result, err := json.MarshalIndent(dataSlice, "", "")
		if err != nil {
			fmt.Println(err)
			os.Exit(1)
		}

		// Write the response to the byte array - Sprintf formats and returns a string without printing it anywhere
		w.Write([]byte(fmt.Sprintf(string(result))))

		// You could also write this response to a file instead of a stream if you wanted to store the data locally...
	})

	// Start the collecting job at the following URL
	c.Visit("https://www.amazon.co.uk/s?k=gopro&ref=nb_sb_noss_2")

}

```

> **Note**: You will notice the import `"github.com/gocolly/colly"`. This is a 3rd party import that has been written to make web scraping much easier.

### Step 2

Now the code is there, you need to add the function as a route on the sever.

Just like you did in [Lab 2](./lab-2.md), you need to create a `http.HandleFunc()` in your `main()` function.

Below the line `http.HandleFunc("/", home)` in your `main()` function in `main.go`, add the line, `http.HandleFunc("/scrape", actions.Scrape)`. Make sure the import for the actions package is also visible. If it is not in the `import` block, then add `"github.com/golang-web-scraping/pkg/actions"` (make sure the system path is correct for your instance).

This line of code will call this function when the route `/scrape` is hit and print out the results.

### Step 3

Run the code in the cloud...

1. Ensure all the code is saved.
2. Ensure you are still signed in to your `ibmcloud` account from the terminal. If your are not logged in, follow the login instructions on [Lab 3](./lab-3.md) step 3.
2. In your terminal, make sure you are in the root directory of your project and enter the command `ibmcloud cf push`. This will push the new code up to the project you already have running in `ibmcloud`.
3. In the URL address bar of a web browser, enter the project address which can be found in the terminal output once the app has started. It will be the value of `routes`. This will show you the home route of the sever.
4. Append `/scrape` to the end of the URL to see the output of the web scraping function.