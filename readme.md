## Organizing Data

Written by Brandon T. Locke; portions from Thomas G. Padilla

### OpenRefine

[OpenRefine](http://openrefine.org/) (formerly known as GoogleRefine), is a very popular tool for working with unorganized, non-normalized (what some may call "messy") data. OpenRefine accepts TSV, CSV, XLS/XLSX, JSON, XML, RDF as XML, and Google Data formats, though others may be used with extensions. It works by opening into your default browser window, but all of the processing takes place on your machine and your data isn't uploaded anywhere.

**This tutorial will demonstrate some of the most popular and powerful features of OpenRefine, including geocoding using an API, algorithmic word normalization and correction, and working with multi-value cells.**

### Data
**You can [download a .zip file of the following three datasets](https://github.com/dmics/organizingdata/blob/master/organizingdata.zip).**

#### Comic Book Metadata (**authors-people.csv**)
Dataset comprising records for comic-books and books relating to comics in the British National Bibliography.[See the full readme](http://www.thomaspadilla.org/data/dataprep/Readme.txt)

#### Academy Award Nominee Metadata (**aa-movies.csv**)
This is a set of data fields scraped from Wikipedia pages for every movie nominated for an Academy Award for best film. Data provided by Taylor Arnold.

#### British Pound to US Dollar Conversion Data (**gbp_conversion.csv**)
This is a GBP to USD conversion rate dataset from 1927 to 2017 downloaded from [MeasuringWorth.com](https://www.measuringworth.com).

### Organizing Comics Metadata
*This portion is adapted from [Thomas Padilla's 'Getting Started with OpenRefine'](http://thomaspadilla.org/dataprep/)*

#### Loading the Dataset
1. Open OpenRefine - it should open a window in your default web browser
1. Click 'Browse' and locate the authors-people.csv on your hard drive. Then click 'Next.'
1. The Configure Parsing Options screen will ask you to confirm a few things. It has made guesses, based on the data, on the type of file, the character encoding and the character that separates columns. Take a look at the data in the top window and make sure everything looks like it's showing up correctly.
1. Name the project "comics-metadata" and click 'Create Project' in the top right corner.

#### Evaluation
Take a minute to look around. Consider the structure of the data with principles of "tidy data" in mind. This will help guide what types of operations you perform on the data. Also take time to evaluate the type of information that is represented and what type of questions you might want to ask of it (e.g. Which publishers are most prominently represented in the collection?)

#### Facet Data
Each column has a facet function that allows you to quickly identify inconsistencies in your data by counting the number of unique occurrences for each piece of data in that column. Using this function we will analyze the distribution of comics in the dataset by publisher. The first step is to begin assessing the consistency of data in the Publisher column.

Try the following sequence of steps:
1. Click the Publisher column > Select Facet > Select Text Facet
2. Facet by 'count' on the left hand side > Observe the record count for the Publisher Titan
3. Facet by 'name' on the left hand side > Scroll down to Titan > Observe variant representation of Titan
4. Hover cursor to the right of 'Titan.,' > Click 'edit' > Type Titan > Titan., transformed to Titan across 33 records

#### Filter Data
Each column contains a 'text filter' function. The text filter is useful for identifying pieces of data that may have many variants. Based on the previous section we know that the publisher Titan may be represented in a variety of ways. In order to explore this further we use the text filter to filter for all occurrences of Titan. Click the Publisher Column > Filter then type 'titan.'

A couple of interesting things result. We see at the top that 4684 rows match Titan. If we were going on the result from step 3 in the prior section we could have assumed that 'Titan' had only 3627 comics in the collection. Closer examination using the text filter function shows us how many records we might have missed as a result of typos and variant spelling.

At this point it is possible to remove some of the observed inconsistency by clicking the edit option within individual variant cells and applying that change to all identical cells.

#### Cluster
In addition to faceting and filtering it is also possible to cluster and normalize variation across the dataset. Clustering will look for patterns of variation without the need for you to (1) sleuth your way through the dataset looking for small variations (2) using facets or filters to eliminate them one at a time. Begin with the default method of 'key collision' using the 'fingerprint' function. Clustering reveals patterns of irregularity throughout the selected column of data. It is then possible to review clustering results and merge the data into the desired form. For more information on all of the available clustering methods and functions consult [OpenRefine documentation on Github](https://github.com/OpenRefine/OpenRefine/wiki/Clustering-In-Depth).

1. Click Publisher > Edit cells > Cluster and edit...
2. Review the proposed merges. Do they make sense?
3. Select a few suggestions that seem right and click 'Merge & Re-Cluster' to edit hundreds of values at once.

#### Transform Data
There are two different ways to transform data using OpenRefine. The first method allows you to utilize preset transformations that perform functions like trimming leading and trailing whitespace (an extra space at the beginning or end of an entry). This might seem like a trivial sort of data formatting issue but without this transformation each piece of data with an extra space at beginning or end would be interpreted as different from an otherwise identical piece of information (e.g. " Titan" and "Titan").

Click Publisher > Edit cells > Common transforms > Trim leading and trailing whitespace

The other method of transformation allows you to utilize the [OpenRefine Expression Language (GREL)](https://github.com/OpenRefine/OpenRefine/wiki/Understanding-Regular-Expressions) to normalize data. Let's say we want to remove the periods in the author names:

1. Click author-persons > Edit cells > Transform...
2. In the GREL window, type `value.replace('.','')`
3. Click ok.
> The replace function searches for the things in the first set of quotes & replaces it with what's in the second set

#### Geocoding
*We'll do a small subset of these cities using Google Maps, which doesn't require a user key to access the geocoder. The downsides of using Google Maps are that they do not allow the data to be used in platforms besides Google Maps, and that they have a limit of 2,500 requests per day. This process is taken from the [OpenRefine Wiki](https://github.com/OpenRefine/OpenRefine/wiki/Geocoding), which also includes instructions on using the Google API in batches to complete an entire dataset. [Click here to see instructions for doing the complete dataset](#full-geocoding-instructions)*

Since there's a limit of 2,500 requests per day and the API takes a bit of time, we'll filter out just two rows to gather latitude and longitude.

- Click Publisher > Facet > Text Facet
- Click on [Associated Newspapers] to limit our set to two rows.
- Click Place of Publication > Edit Column > Add Column by Fetching URLs... and enter this expression: `"http://maps.google.com/maps/api/geocode/json?sensor=false&address=" + escape(value, "url")`
- Name the column 'geocodingResponse', add a 0 to the 'Throttle delay,' and click OK. This will take 20-30 seconds to finish.
- The new 'geocodingResponse' column won't be very clear or useful - it will be the full JSON response with all of the information Google has about that location.
- Click geocodingResponse > Edit Column > Add Column based on this column
- Enter `with(value.parseJson().results[0].geometry.location, pair, pair.lat +", " + pair.lng)` and call the new column 'latlng.' Hit OK. This will parse the JSON and correctly format the latitute and longitude in the new column.
- You can delete the 'geocodingResponse' column (Edit Column > Remove This Column) after you have already extracted the lat/lng coordinates.

> Some software will want latitude and longitude separately. If that's the case, Edit latlng Column > Split into several columns... and then split by the substring ","

#### Saving and Exporting
In the top right corner, you can click on 'Export' and save the data in a number of different formats, including csv and HTML tables.

You may also want to export the entire project. This is useful if you want to share the project with others, or if you want to continue working on a different machine. It's also useful for transparency and documentation, as every change you've made is documented (and reversible).

#### Full Geocoding Instructions

*[Geocod.io](https://geocod.io/) may also be a good option*

*Note: this will take several hours to process fully, so it's a good idea to set it up to run overnight*

- Get a MapQuest API Key from the [MapQuest Developer Site](https://developer.mapquest.com/) - click the 'Get your Free API Key' button on the front page and fill out the information.
- Once you have an API key, Location > Edit Column > Add Column by Fetching URLs... and enter this expression: `'http://open.mapquestapi.com/nominatim/v1/search.php?' + 'key=YOUR KEY&' + 'format=json&' + 'q=' + escape(value, 'url')` **Note: be sure to add your own API key in the above expression where it says `*YOUR KEY*`**
- Name the column 'geocodingResponse' and click OK. This will take quite some time to finish.
- The new 'geocodingResponse' column won't be very clear or useful - it will be the full JSON response with all of the information Google has about that location.
- Click geocodingResponse > Edit Column > Add Column based on this column
- Enter `with(value.parseJson().resourceSets[0].resources[0].point.coordinates, pair, pair[0] +", " + pair[1])` and call the new column 'latlng.' Hit OK. This will parse the JSON and correctly format the latitute and longitude in the new column.
- You should see that the resulting column has the latitude and longitude for the address or cross streets.

### Organizing Movie metadata

#### Loading the Dataset
1. Click 'Open' in the top right to open a new OpenRefine tab
1. Click 'Browse' and locate the aa-movies.csv on your hard drive. Then click 'Next.'
1. The Configure Parsing Options screen will ask you to confirm a few things. It has made guesses, based on the data, on the type of file, the character encoding and the character that separates columns. Take a look at the data in the top window and make sure everything looks like it's showing up correctly.
1. Name the project "movies-metadata" and click 'Create Project' in the top right corner.

#### Evaluation
Take a minute to look around. Consider the structure of the data with principles of "tidy data" in mind. This will help guide what types of operations you perform on the data. Also take time to evaluate the type of information that is represented and what type of questions you might want to ask of it (e.g. Which publishers are most prominently represented in the collection?)

#### Normalizing Monetary Info by Joining a New Dataset
1. One thing we may notice right away is that some most of the budget and box_office numbers are in USD, but not all of them are.
1. Before we do anything, we'll want to make sure that our two numbered columns—budget & box_office—are being recognized as numbers and not strings. OpenRefine shows numbers in green, and also aligns them on the right side of the column.
1. To change a column from a string to a number, click on the column header > Edit Cells > Common transforms > To numbers.
1. Click 'Open' in the top right - this will open a new Open Refine tab
2. Load 'gbp_conversion.csv' and create a project called 'conversion'
3. Go back to the movies-metadata project, click Year > Create column based on this column.
4. Call the new column 'GPBperUSD' and then in the GREL window, type `cell.cross("conversion", "Year")[0].cells["GBPperUSD"].value`. This looks to the "conversion" project, looks at the "Year" column in it, matches it to the "year" column in our project, and then pulls the "GBPperUSD" into this new column. Click OK.
5.click budget, transform
6. in the GREL box, type `value * cells["GBPperUSD"].value` to take the budget column and multiply it by the conversion rate. Click OK.
7. Now click on the EUR facet. Since there's just one in this dataset, it's easier to do this manually than to join a whole new dataset. The Euro to USD rate was about 1.33 in 2014. Select budget > Edit Cells > Transform and then enter `value * 1.33`. Click OK.
8. Click on 'Remove All' on the left to remove our budget facet and bring back the entire dataset. Now that all of our budget numbers are in USD, let's remove the budget_currency column. budget_currency > Edit column > Remove this column.
9. Do the same for the box office numbers. box_office_currency > Facet > Text facet.
10. Select GBP.
11. box_office > Edit cells > Transform. Enter `value * cells["GBPperUSD"].value`. Click OK.
12. box_office_currency > Edit column > Remove this column. Repeat for 'GBPperUSD'

#### Working with Multi-Value Cells
This data has multiple actors listed in the 'Starring' column, with each name separated by a comma. For many purposes, this works great—it's the most compact and concise way to represent this information. But for many purposes, you may need to format this data differently. Here are a few different ways you may need to export this dataset.

**Multiple 'Starring' Columns with One Person in Each**
1. Select Starring > Edit Column > Split into several columns
2. You'll see a few options—for this you want to leave things as-is and separate on the commas. Click OK.
3. You now have 19 separate 'Starring' columns, and each column is either empty or has one value.

Let's try another way. Click on 'Undo/Redo' in the top left and undo your split command.

**Create TRUE/FALSE Columns Based on Values**
1. Select Starring > Edit column based on this column
2. In the GREL window, type `if(value.contains("Marie Prevost"), "true", "false")`
3. Call this column 'isMariePrevost' and click OK.
4. Do this again for Thomas Meighan. Starring > Edit column based on this column.
5. Type `if(value.contains("Thomas Meighan"), "true", "false")`, name it 'isThomasMeighan' and click OK.

**Create an Actor-Movie Network Dataset**
To create networks, we need to have source and target pairs—in other words, we need a separate line with each unique combination of movie and actor.

1. Select Starring > Edit cells > Split multi-valued cells
2. This will open a dialog box asking for a separator—leave the comma and click OK.
3. You'll see that we've now added several rows below each row with multiple actors listed. The actor name is the only bit of information in the entire row.
4. Select movie_id > Edit cells > Fill down.
5. You can do this for each column of information that you want in the network dataset.


## Additional OpenRefine Resources
- [OpenRefine Wiki](https://github.com/OpenRefine/OpenRefine/wiki)
- [OpenRefine Recipes](https://github.com/OpenRefine/OpenRefine/wiki/Recipes)
- [Cleaning Data with OpenRefine](https://libjohn.github.io/openrefine/)
- [Fetching and Parsing Data from the Web with OpenRefine](https://programminghistorian.org/lessons/fetch-and-parse-data-with-openrefine)
