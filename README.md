# UK Food Hygiene Ratings Analysis (nosql-challenge)

This project involves evaluating and wrangling data from the UK Food Standards Agency. The data is used to assist the editors of the food magazine, *Eat Safe, Love*, in identifying establishments that deserve attention in future articles. The project is divided into three main parts: Database Setup, Database Update, and Exploratory Analysis.

## Part 1: Database and Jupyter Notebook Set Up
1. **Import the Data**: Use the `establishments.json` file provided.
   - **Database Name**: `uk_food`
   - **Collection Name**: `establishments`

   ```sh
   mongoimport --db uk_food --collection establishments --file establishments.json --jsonArray
   ```

### Part 2: Update the Database
2. Add a New Restaurant
   **Create a variable of the new resturant and add it to the collection**:
   ```
   new_resturant = {
    "BusinessName":"Penang Flavours",
    "BusinessType":"Restaurant/Cafe/Canteen",
    "BusinessTypeID":"",
    "AddressLine1":"Penang Flavours",
    "AddressLine2":"146A Plumstead Rd",
    "AddressLine3":"London",
    "AddressLine4":"",
    "PostCode":"SE18 7DY",
    "Phone":"",
    "LocalAuthorityCode":"511",
    "LocalAuthorityName":"Greenwich",
    "LocalAuthorityWebSite":"http://www.royalgreenwich.gov.uk",
    "LocalAuthorityEmailAddress":"health@royalgreenwich.gov.uk",
    "scores":{
        "Hygiene":"",
        "Structural":"",
        "ConfidenceInManagement":""
    },
    "SchemeType":"FHRS",
    "geocode":{
        "longitude":"0.08384000",
        "latitude":"51.49014200"
    },
    "RightToReply":"",
    "Distance":4623.9723280747176,
    "NewRatingPending":True
   }
   
   establishments.insert_one(new_resturant)
   ```
3. Remove Dover Establishments
   ```
   establishments.delete_many({'LocalAuthorityName': 'Dover'})
   ```
4. Data Type Corrections
   **Change the data type from String to Decimal for longitude and latitude**:
   ```
   establishments.update_many({}, [{'$set': {'geocode.longitude': {'$toDouble': '$geocode.longitude'}, 
                                         'geocode.latitude': {'$toDouble': '$geocode.latitude'}
                                         }}])
   ```
   **Set non 1-5 Rating Values to Null and change the data type from String to Integer for RatingValue**:
   ```
   non_ratings = ["AwaitingInspection", "Awaiting Inspection", "AwaitingPublication", "Pass", "Exempt"]
   establishments.update_many({"RatingValue": {"$in": non_ratings}}, [ {'$set':{ "RatingValue" : None}} ])

   establishments.update_many({"RatingValue": {"$type": "string"}},
    [{"$set": {"RatingValue": {"$toInt": "$RatingValue"}}}]
   )
   ```
### Part 3: Exploratory Analysis
1. Which establishments have a hygiene score equal to 20?
   ```
    query = {'scores.Hygiene': 20}
   hygiene_count = establishments.count_documents(query)
   hygiene_count
   ```
2. Which establishments in London have a RatingValue greater than or equal to 4?
   ```
   query = {'LocalAuthorityName': {'$regex': 'London'},
         'RatingValue': {'$gte': 4 }}
   
   local_rating_count = establishments.count_documents(query)
   local_rating_count
   ```
3. What are the top 5 establishments with a RatingValue rating value of 5, sorted by lowest hygiene score, nearest to the new restaurant added, "Penang Flavours"?
   ```
   degree_search = 0.01
   latitude = 51.49014200
   longitude = 0.08384000

   query ={
    'RatingValue': 5,
    'geocode.latitude':{
        '$gte': latitude-degree_search,
        '$lte': latitude+degree_search
    },
    'geocode.longitude': {
        '$gte': longitude-degree_search,
        '$lte': longitude+degree_search
    }
   }
   sort = [('score.Hygiene', 1)]
   limit = 5

   pprint(list(establishments.find(query).sort(sort). limit(limit)))
   ```
4. How many establishments in each Local Authority area have a hygiene score of 0?
    ```
    pipeline = [
    {'$match': {'scores.Hygiene': 0}},
    {
        '$group': {
            '_id': '$LocalAuthorityName',
    'count': {'$sum': 1}
            }
    },
    {'$sort': {'count': -1}}
    ]
    result = list(establishments.aggregate(pipeline))
    ```
