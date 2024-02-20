# Analyzing Mobile App Data

In this analysis, we will be looking into a dataset of mobile apps for IOS/Android. We will be looking at apps that are free to download/install on Google Play and the App Store. We will identify which genres and categories are the most popular, and our ultimate goal is to gather insights on the specific genre/category of apps that will attract the most number of users.

I'll be approaching this research as if I am an analyst at a company, and I am investigating on their behalf to determine what kind of app the company should create.

### Opening and Exploring the Data ###

The dataset that will be analyzed is a sample that contains data of about 10,000 Android apps from Google Play, and 7,000 iOS apps from the App Store. The dataset can be accessed through the following links: 

- <a href="https://www.kaggle.com/datasets/lava18/google-play-store-apps">Google Play</a>
- <a href="https://www.kaggle.com/datasets/ramamet4/app-store-apple-data-set-10k-apps">App Store</a>

In this analysis, we will focus only on apps that are **free** and use **English** as its main language. 


```python
##import 'reader' class from the 'csv' module
from csv import reader


##ios
opened_file = open('AppleStore.csv') 
read_file = reader(opened_file)     ##use the reader to open and read file
ios = list(read_file)               ##convert the csv file to a list of lists
ios_header = ios[0]                 ##extract the header row
ios = ios[1:]                       ##splice out the header row

##android
opened_file = open('googleplaystore.csv')
read_file = reader(opened_file)
android = list(read_file)
android_header = android[0]
android = android[1:]


```

We begin by opening the dataset files and converting them to a list of lists. We then remove the header rows to make it easier for Python to skip over the headers when analyzing the data. We can now start exploring our datasets. We will begin with the Google Play data. 


```python
def explore_data(dataset, start, end, rows_and_columns=False):
    dataset_slice = dataset[start:end]    
    for row in dataset_slice:
        print(row)
        print('\n') # adds a new (empty) line between rows
        
    if rows_and_columns:
        print('Number of rows:', len(dataset))
        print('Number of columns:', len(dataset[0]))

print(android_header)
print('\n')
explore_data(android, 0, 3, True)
```

    ['App', 'Category', 'Rating', 'Reviews', 'Size', 'Installs', 'Type', 'Price', 'Content Rating', 'Genres', 'Last Updated', 'Current Ver', 'Android Ver']
    
    
    ['Photo Editor & Candy Camera & Grid & ScrapBook', 'ART_AND_DESIGN', '4.1', '159', '19M', '10,000+', 'Free', '0', 'Everyone', 'Art & Design', 'January 7, 2018', '1.0.0', '4.0.3 and up']
    
    
    ['Coloring book moana', 'ART_AND_DESIGN', '3.9', '967', '14M', '500,000+', 'Free', '0', 'Everyone', 'Art & Design;Pretend Play', 'January 15, 2018', '2.0.0', '4.0.3 and up']
    
    
    ['U Launcher Lite – FREE Live Cool Themes, Hide Apps', 'ART_AND_DESIGN', '4.7', '87510', '8.7M', '5,000,000+', 'Free', '0', 'Everyone', 'Art & Design', 'August 1, 2018', '1.2.4', '4.0.3 and up']
    
    
    Number of rows: 10841
    Number of columns: 13


A function called `explore_data` was written that will explore the rows and columnms of the data sets in a more readable way. The function prints the first few rows of the dataset and also gives us the number of rows and columns. The first row that the function gives is the header row that tells us the column labels. Among the columns, the `App`, `Category`, `Rating`, `Reviews`, `Installs`, `Type`, `Price`, `Content Rating`, and `Genres` columns are the ones that should be noted, as these columns may help us understand and gauge the popularity of certain apps. We can also see that the Google Play data set has 10841 rows and 13 columns.

Now, let's explore the App Store data in the same way. 


```python
print(ios_header)
print('\n')
explore_data(ios, 0, 3, True)
```

    ['id', 'track_name', 'size_bytes', 'currency', 'price', 'rating_count_tot', 'rating_count_ver', 'user_rating', 'user_rating_ver', 'ver', 'cont_rating', 'prime_genre', 'sup_devices.num', 'ipadSc_urls.num', 'lang.num', 'vpp_lic']
    
    
    ['284882215', 'Facebook', '389879808', 'USD', '0.0', '2974676', '212', '3.5', '3.5', '95.0', '4+', 'Social Networking', '37', '1', '29', '1']
    
    
    ['389801252', 'Instagram', '113954816', 'USD', '0.0', '2161558', '1289', '4.5', '4.0', '10.23', '12+', 'Photo & Video', '37', '0', '29', '1']
    
    
    ['529479190', 'Clash of Clans', '116476928', 'USD', '0.0', '2130805', '579', '4.5', '4.5', '9.24.12', '9+', 'Games', '38', '5', '18', '1']
    
    
    Number of rows: 7197
    Number of columns: 16


The App Store dataset has 7197 rows and 16 columns. Among the columns, the `price`, `rating_count_tot`, `rating_count_ver`, `user_rating`, and `prime_genre` are likely columns that are relevant in analyzing the popularity of apps. We will keep these columns in mind as we proceed with our analysis. 

### Deleting Wrong Data ###

This next part of the analysis deals with data cleaning. Datasets often contain inaccurate and duplicate data. In order to specifically target apps that are free and uses English, we must remove non-English Apps and remove apps that aren't free. In addition, if there is duplicate data, we must detect and remove it. 

In the Google Play dataset's discussion section, a user noted one example of an inaccurate data. The app in row 10472 of the dataset has a rating of 19, when the maximum possible rating for an app on Google Play is 4. We will simply delete this row and move on.


```python
print(android[10472])
print('\n')
print(android_header)  # header

print(len(android))
del android[10472]  
print(len(android))
```

    ['Life Made WI-Fi Touchscreen Photo Frame', '1.9', '19', '3.0M', '1,000+', 'Free', '0', 'Everyone', '', 'February 11, 2018', '1.0.19', '4.0 and up']
    
    
    ['App', 'Category', 'Rating', 'Reviews', 'Size', 'Installs', 'Type', 'Price', 'Content Rating', 'Genres', 'Last Updated', 'Current Ver', 'Android Ver']
    10841
    10840


The name of the app was *Life Made Wi-Fi Touchscreen Photo Frame* and we can confirm that it has an error rating of 19. The row has been deleted and we can see that instead of 10841 rows, the dataset now has 10840. 

### Removing Duplicate Entries: Part One ###

The Google Play data set has lots of duplicate entries. We'll be removing rows based on how recent the data entry is. By looking at the reviews for each app, we can know how recent the data is. For example, out of the four duplicate data entries for Instagram, the one with the most reviews will be the most recent data. We will keep that one and remove the other 3.

Instagram is an example of an app in Google Play that has duplicate entries. It occurs 4 times


```python
for app in android: 
    name = app[0]
    if name == 'Instagram':
        print(app)
```

    ['Instagram', 'SOCIAL', '4.5', '66577313', 'Varies with device', '1,000,000,000+', 'Free', '0', 'Teen', 'Social', 'July 31, 2018', 'Varies with device', 'Varies with device']
    ['Instagram', 'SOCIAL', '4.5', '66577446', 'Varies with device', '1,000,000,000+', 'Free', '0', 'Teen', 'Social', 'July 31, 2018', 'Varies with device', 'Varies with device']
    ['Instagram', 'SOCIAL', '4.5', '66577313', 'Varies with device', '1,000,000,000+', 'Free', '0', 'Teen', 'Social', 'July 31, 2018', 'Varies with device', 'Varies with device']
    ['Instagram', 'SOCIAL', '4.5', '66509917', 'Varies with device', '1,000,000,000+', 'Free', '0', 'Teen', 'Social', 'July 31, 2018', 'Varies with device', 'Varies with device']


Let's check to see how many duplicate apps we have total in Google Play:


```python
duplicate_apps = []
unique_apps = []

for app in android: 
    name = app[0]
    if name in unique_apps: 
        duplicate_apps.append(name)
    else: 
        unique_apps.append(name)
        
print('Number of duplicate apps:', len(duplicate_apps))
print('\n')
print('Examples of duplicate apps:', duplicate_apps[:15])
```

    Number of duplicate apps: 1181
    
    
    Examples of duplicate apps: ['Quick PDF Scanner + OCR FREE', 'Box', 'Google My Business', 'ZOOM Cloud Meetings', 'join.me - Simple Meetings', 'Box', 'Zenefits', 'Google Ads', 'Google My Business', 'Slack', 'FreshBooks Classic', 'Insightly CRM', 'QuickBooks Accounting: Invoicing & Expenses', 'HipChat - Chat Built for Teams', 'Xero Accounting Software']


In total, there are 1181 apps in Google Play that have duplicate entries. We must find a way to remove these duplicate entries and make sure there is only one entry per app. For each app, we will keep the entry that has the highest number of reviews and delete the other entries.

In order to do this, we will build a dictionary called `reviews_max`. Then, we will loop through the android data set and add each entry to `reviews_max dictionary`. If the entry is already in `reviews_max`, the code checks to see if the current entry has a higher number of reviews than the one in the dictionary. The entry with the higher number of reviews is kept. The dictionary will only contain the maximum number of reviews for each app in the dataset. 

### Removing Duplicate Entries: Part Two ###


```python
reviews_max = {}

for app in android:
    name = app[0]
    n_reviews = float(app[3])
    
    if name in reviews_max and reviews_max[name] < n_reviews:
        reviews_max[name] = n_reviews
    elif name not in reviews_max:
        reviews_max[name] = n_reviews


print('Expected length:', len(android) - 1181)
print('Actual length:', len(reviews_max))
```

    Expected length: 9659
    Actual length: 9659


The code loops through the dataset and uses the `reviews_max` dictionary as a reference to determine if the data entry that is being examined has the highest number of reviews among its duplicates. If the number of reviews for the data entry matches that of the number of reviews in "reviews_max," this means that it has the highest number of reviews. 



```python
android_clean = []
already_added = []

for app in android: 
    name = app[0]
    n_reviews = float(app[3])
    
    if (reviews_max[name] == n_reviews) and (name not in already_added):
        android_clean.append(app)
        already_added.append(name)

```

To remove the duplicate entries, two empty lists were made: `android_clean` and `already_added`. Then we loop through the data set and identify the app name and number of reviews. If the number of reviews of the app matches the number of reviews in the `reviews_max` dictionary, then it will be appended to `android_clean` and the app name to the `already_added` list. The second part of the conditional addresses the possibility where the highest number of reviews of a duplicate app is the same for more than one entry. 


```python
explore_data(android_clean, 0, 4, True)
```

    ['Photo Editor & Candy Camera & Grid & ScrapBook', 'ART_AND_DESIGN', '4.1', '159', '19M', '10,000+', 'Free', '0', 'Everyone', 'Art & Design', 'January 7, 2018', '1.0.0', '4.0.3 and up']
    
    
    ['U Launcher Lite – FREE Live Cool Themes, Hide Apps', 'ART_AND_DESIGN', '4.7', '87510', '8.7M', '5,000,000+', 'Free', '0', 'Everyone', 'Art & Design', 'August 1, 2018', '1.2.4', '4.0.3 and up']
    
    
    ['Sketch - Draw & Paint', 'ART_AND_DESIGN', '4.5', '215644', '25M', '50,000,000+', 'Free', '0', 'Teen', 'Art & Design', 'June 8, 2018', 'Varies with device', '4.2 and up']
    
    
    ['Pixel Draw - Number Art Coloring Book', 'ART_AND_DESIGN', '4.3', '967', '2.8M', '100,000+', 'Free', '0', 'Everyone', 'Art & Design;Creativity', 'June 20, 2018', '1.1', '4.4 and up']
    
    
    Number of rows: 9659
    Number of columns: 13


Upon using the `explore_data` function, it is confirmed that the `android_clean` list contains 9659 rows. Therefore, we can safely assume that duplicate entries have been removed and this list contains only unique data entries. 

### Removing Non-English Apps ###

The dataset also has some non-English apps that needs to be removed. Here are some examples:


```python
print(ios[813][1])
print(ios[6731][1])
print('\n')
print(android_clean[4412][0])
print(android_clean[7940][0])
```

    爱奇艺PPS -《欢乐颂2》电视剧热播
    【脱出ゲーム】絶対に最後までプレイしないで 〜謎解き＆ブロックパズル〜
    
    
    中国語 AQリスニング
    لعبة تقدر تربح DZ


To remove such non-English apps, we first need a function that can detect non-English letters. The function `is_it_english` below takes in each character in a string and returns True or False depending on whether it corresponds to the ASCII (American Standard Code for Information Interchange) system. 


```python
def is_it_english(string):
    non_ASCII = 0 
    
    for character in string:
        if ord(character) > 127:
            non_ASCII += 1
        
    if non_ASCII > 3:
        return False
    else: 
        return True 
    
print(is_it_english('Instagram'))
print(is_it_english('爱奇艺PPS -《欢乐颂2》电视剧热播'))
print(is_it_english('Docs To Go™ Free Office Suite'))
print(is_it_english('Instachat 😜'))
```

    True
    False
    True
    True


We can see that the function is working as intended. If an app is in another language, the function will return False. Emojis and trademarks are fine and will return as True as long as the rest of the title is in English. Next, we will apply this function as a filter as we gather all the English language apps into a new list. 


```python
android_english = []
ios_english = []

for app in android_clean: 
    name = app[0]
    if is_it_english(name):
        android_english.append(app)
        
for app in ios:
    name = app[1]
    if is_it_english(name):
        ios_english.append(app)
        
explore_data(android_english, 0, 3, True)
print('\n')
explore_data(ios_english, 0, 3, True)
```

    ['Photo Editor & Candy Camera & Grid & ScrapBook', 'ART_AND_DESIGN', '4.1', '159', '19M', '10,000+', 'Free', '0', 'Everyone', 'Art & Design', 'January 7, 2018', '1.0.0', '4.0.3 and up']
    
    
    ['U Launcher Lite – FREE Live Cool Themes, Hide Apps', 'ART_AND_DESIGN', '4.7', '87510', '8.7M', '5,000,000+', 'Free', '0', 'Everyone', 'Art & Design', 'August 1, 2018', '1.2.4', '4.0.3 and up']
    
    
    ['Sketch - Draw & Paint', 'ART_AND_DESIGN', '4.5', '215644', '25M', '50,000,000+', 'Free', '0', 'Teen', 'Art & Design', 'June 8, 2018', 'Varies with device', '4.2 and up']
    
    
    Number of rows: 9614
    Number of columns: 13
    
    
    ['284882215', 'Facebook', '389879808', 'USD', '0.0', '2974676', '212', '3.5', '3.5', '95.0', '4+', 'Social Networking', '37', '1', '29', '1']
    
    
    ['389801252', 'Instagram', '113954816', 'USD', '0.0', '2161558', '1289', '4.5', '4.0', '10.23', '12+', 'Photo & Video', '37', '0', '29', '1']
    
    
    ['529479190', 'Clash of Clans', '116476928', 'USD', '0.0', '2130805', '579', '4.5', '4.5', '9.24.12', '9+', 'Games', '38', '5', '18', '1']
    
    
    Number of rows: 6183
    Number of columns: 16


Upon filtering out the non-English apps, we are left with 9614 android apps and 6183 iOS apps. 

### Isolating the Free Apps ###

Now that we have made sure we are only working with English-language apps, the next step is to isolate the free apps. The dataset currently consists of both free and paid apps. The cell below runs a loop to detect apps that have a price of 0 and appends those apps into the `android_free` and `ios_free` lists. 


```python
android_free = []
ios_free = []

for app in android_english: 
    price = app[7]
    if price == '0':
        android_free.append(app)
        
for app in ios_english:
    price = app[4]
    if price == '0.0':
        ios_free.append(app)

print(len(android_free))
print(len(ios_free))
```

    8864
    3222


There are 8864 free apps on Google Play and 3222 free apps on the App Store. These are the final lists of apps that will be analyzed in this investigation. 

### Analyzing the Most Common Apps by Genre ###

The data cleaning process is complete and we can finally start analyzing the data that we have organized and sorted. We will first analyze how common an app genre is by looking at how much representation they have in their mobile platforms. Then, we will gauge their popularity by looking at how many number of users each app genre has. 

In order to visualize the data that we have, a frequency table must be constructed. The code below constructs a frequency table that displays the percentages of each genre of apps in each mobile platform and displays the percentages in a descending order. 



```python
def freq_table(dataset, index):
    table = {}
    total = 0
    
    for row in dataset:
        total += 1
        value = row[index]
        if value in table:
            table[value] += 1
        else:
            table[value] = 1
    
    table_percentages = {}
    for key in table:
        percentage = (table[key] / total) * 100
        table_percentages[key] = percentage 
    
    return table_percentages

            
    
def display_table(dataset, index):
    table = freq_table(dataset, index)
    table_display = []
    for key in table:
        key_val_as_tuple = (table[key], key)
        table_display.append(key_val_as_tuple)

    table_sorted = sorted(table_display, reverse = True)
    for entry in table_sorted:
        print(entry[1], ':', entry[0])
```

Now, we will analyze the apps and gauge how common each app genre is based on how common they are in their respective platforms. 

**App Store**


```python
display_table(ios_free, -5)
```

    Games : 58.16263190564867
    Entertainment : 7.883302296710118
    Photo & Video : 4.9658597144630665
    Education : 3.662321539416512
    Social Networking : 3.2898820608317814
    Shopping : 2.60707635009311
    Utilities : 2.5139664804469275
    Sports : 2.1415270018621975
    Music : 2.0484171322160147
    Health & Fitness : 2.0173805090006205
    Productivity : 1.7380509000620732
    Lifestyle : 1.5828677839851024
    News : 1.3345747982619491
    Travel : 1.2414649286157666
    Finance : 1.1173184357541899
    Weather : 0.8690254500310366
    Food & Drink : 0.8069522036002483
    Reference : 0.5586592178770949
    Business : 0.5276225946617008
    Book : 0.4345127250155183
    Navigation : 0.186219739292365
    Medical : 0.186219739292365
    Catalogs : 0.12414649286157665


**App Store Analysis**

Based on this data, the most common app category in the App Store is the gaming genre. Entertainment apps come at second place, but the competition is not close. Games are by far the most dominant category. 

On a broader scale, apps in the App Store that are designed for entertainment purposes (such as games, social media, photography, etc.) generally tend to be more common than apps designed for practical purposes (such as education, shopping, utilities, productivity, etc.). However, it is important to note that the broad availability of apps of a certain category does not automatically translate to popularity. It is entirely possible that the App Store is bombarded by game apps that nobody wants to play. The availability of theses apps may not necessarily match the demand for them and we will keep this in mind as we proceed with our analysis. 

**Google Play**


```python
display_table(android_free, 1)
```

    FAMILY : 18.907942238267147
    GAME : 9.724729241877256
    TOOLS : 8.461191335740072
    BUSINESS : 4.591606498194946
    LIFESTYLE : 3.9034296028880866
    PRODUCTIVITY : 3.892148014440433
    FINANCE : 3.7003610108303246
    MEDICAL : 3.531137184115524
    SPORTS : 3.395758122743682
    PERSONALIZATION : 3.3167870036101084
    COMMUNICATION : 3.2378158844765346
    HEALTH_AND_FITNESS : 3.0798736462093865
    PHOTOGRAPHY : 2.944494584837545
    NEWS_AND_MAGAZINES : 2.7978339350180503
    SOCIAL : 2.6624548736462095
    TRAVEL_AND_LOCAL : 2.33528880866426
    SHOPPING : 2.2450361010830324
    BOOKS_AND_REFERENCE : 2.1435018050541514
    DATING : 1.861462093862816
    VIDEO_PLAYERS : 1.7937725631768955
    MAPS_AND_NAVIGATION : 1.3989169675090252
    FOOD_AND_DRINK : 1.2409747292418771
    EDUCATION : 1.1620036101083033
    ENTERTAINMENT : 0.9589350180505415
    LIBRARIES_AND_DEMO : 0.9363718411552346
    AUTO_AND_VEHICLES : 0.9250902527075812
    HOUSE_AND_HOME : 0.8235559566787004
    WEATHER : 0.8009927797833934
    EVENTS : 0.7107400722021661
    PARENTING : 0.6543321299638989
    ART_AND_DESIGN : 0.6430505415162455
    COMICS : 0.6204873646209386
    BEAUTY : 0.5979241877256317


**Google Play Analysis** 

In the Google Play Store, the most common app genre is "Family." It is almost double that of the next most common category, which is the "Game" genre. "Tools" genre is a close third place. 

Compared to the App Store data, games are still among the most common apps in Google Play. In fact, the Family genre is composed mostly of games meant for kids. However, they are nowhere near as common as they are in the App Store. It looks like the apps that are geared towards families tend to be more common. 

The dataset here is definitely less skewed towards a single category as it is for the gaming category in the App Store. There is much better representation of practical apps in the Google Play Store. 

### Most Popular Apps by Genre (App Store) ###

So far, we have analyzed the app genres by how common and how much representation they have in their respective platforms. Now, we will determine the most popular genre of apps by looking at the number of users. The App Store data does not provide us with the number of users, so we are going to use the total number of user ratings instead. 

The code below is similar to what we did when analyzing the most common apps. The frequency table will generate the number of users for each genre. 


```python
genres_ios = freq_table(ios_free, -5)

# Create a list to store tuples of (average_n_ratings, genre)
avg_ratings_by_genre = []

for genre in genres_ios:
    total = 0
    len_genre = 0
    for app in ios_free:
        genre_app = app[-5]
        if genre_app == genre:
            n_ratings = float(app[5])
            total += n_ratings
            len_genre += 1
    if len_genre > 0:
        average_n_ratings = total / len_genre
        avg_ratings_by_genre.append((average_n_ratings, genre))

# Sort the list of tuples in descending order based on average ratings
avg_ratings_by_genre.sort(reverse=True)

# Print the sorted results
for avg_rating, genre in avg_ratings_by_genre:
    print(genre, ":", avg_rating)
```

    Navigation : 86090.33333333333
    Reference : 74942.11111111111
    Social Networking : 71548.34905660378
    Music : 57326.530303030304
    Weather : 52279.892857142855
    Book : 39758.5
    Food & Drink : 33333.92307692308
    Finance : 31467.944444444445
    Photo & Video : 28441.54375
    Travel : 28243.8
    Shopping : 26919.690476190477
    Health & Fitness : 23298.015384615384
    Sports : 23008.898550724636
    Games : 22788.6696905016
    News : 21248.023255813954
    Productivity : 21028.410714285714
    Utilities : 18684.456790123455
    Lifestyle : 16485.764705882353
    Entertainment : 14029.830708661417
    Business : 7491.117647058823
    Education : 7003.983050847458
    Catalogs : 4004.0
    Medical : 612.0


Based on reviews alone, navigations apps have the highest number of users. However, we must be cautious before determining that navigation apps are the most popular genre in the App Store. If we look into the data closely, it is clear that most of these reviews are exclusively for only 2 specific apps: Google Maps and Waze. 


```python
for app in ios_free:
    if app[-5] == 'Navigation':
        print(app[1], ':', app[5]) # print name and number of ratings
```

    Waze - GPS Navigation, Maps & Real-time Traffic : 345046
    Google Maps - Navigation & Transit : 154911
    Geocaching® : 12811
    CoPilot GPS – Car Navigation & Offline Maps : 3582
    ImmobilienScout24: Real Estate Search in Germany : 187
    Railway Route Search : 5


Google Maps and Waze alone make up for almost 500,000 reviews for navigation apps. It's important to keep in mind that when it comes to certain genres that have a lot of reviews, some of these numbers are skewed by a few mainstream apps that happen to to dominate the scene. Here are some examples: 




```python
for app in ios_free:
    if app[-5] == 'Reference':
        print(app[1], ':', app[5]) # print name and number of ratings
```

    Bible : 985920
    Dictionary.com Dictionary & Thesaurus : 200047
    Dictionary.com Dictionary & Thesaurus for iPad : 54175
    Google Translate : 26786
    Muslim Pro: Ramadan 2017 Prayer Times, Azan, Quran : 18418
    New Furniture Mods - Pocket Wiki & Game Tools for Minecraft PC Edition : 17588
    Merriam-Webster Dictionary : 16849
    Night Sky : 12122
    City Maps for Minecraft PE - The Best Maps for Minecraft Pocket Edition (MCPE) : 8535
    LUCKY BLOCK MOD ™ for Minecraft PC Edition - The Best Pocket Wiki & Mods Installer Tools : 4693
    GUNS MODS for Minecraft PC Edition - Mods Tools : 1497
    Guides for Pokémon GO - Pokemon GO News and Cheats : 826
    WWDC : 762
    Horror Maps for Minecraft PE - Download The Scariest Maps for Minecraft Pocket Edition (MCPE) Free : 718
    VPN Express : 14
    Real Bike Traffic Rider Virtual Reality Glasses : 8
    教えて!goo : 0
    Jishokun-Japanese English Dictionary & Translator : 0



```python
for app in ios_free:
    if app[-5] == 'Social Networking':
        print(app[1], ':', app[5]) # print name and number of ratings
```

    Facebook : 2974676
    Pinterest : 1061624
    Skype for iPhone : 373519
    Messenger : 351466
    Tumblr : 334293
    WhatsApp Messenger : 287589
    Kik : 260965
    ooVoo – Free Video Call, Text and Voice : 177501
    TextNow - Unlimited Text + Calls : 164963
    Viber Messenger – Text & Call : 164249
    Followers - Social Analytics For Instagram : 112778
    MeetMe - Chat and Meet New People : 97072
    We Heart It - Fashion, wallpapers, quotes, tattoos : 90414
    InsTrack for Instagram - Analytics Plus More : 85535
    Tango - Free Video Call, Voice and Chat : 75412
    LinkedIn : 71856
    Match™ - #1 Dating App. : 60659
    Skype for iPad : 60163
    POF - Best Dating App for Conversations : 52642
    Timehop : 49510
    Find My Family, Friends & iPhone - Life360 Locator : 43877
    Whisper - Share, Express, Meet : 39819
    Hangouts : 36404
    LINE PLAY - Your Avatar World : 34677
    WeChat : 34584
    Badoo - Meet New People, Chat, Socialize. : 34428
    Followers + for Instagram - Follower Analytics : 28633
    GroupMe : 28260
    Marco Polo Video Walkie Talkie : 27662
    Miitomo : 23965
    SimSimi : 23530
    Grindr - Gay and same sex guys chat, meet and date : 23201
    Wishbone - Compare Anything : 20649
    imo video calls and chat : 18841
    After School - Funny Anonymous School News : 18482
    Quick Reposter - Repost, Regram and Reshare Photos : 17694
    Weibo HD : 16772
    Repost for Instagram : 15185
    Live.me – Live Video Chat & Make Friends Nearby : 14724
    Nextdoor : 14402
    Followers Analytics for Instagram - InstaReport : 13914
    YouNow: Live Stream Video Chat : 12079
    FollowMeter for Instagram - Followers Tracking : 11976
    LINE : 11437
    eHarmony™ Dating App - Meet Singles : 11124
    Discord - Chat for Gamers : 9152
    QQ : 9109
    Telegram Messenger : 7573
    Weibo : 7265
    Periscope - Live Video Streaming Around the World : 6062
    Chat for Whatsapp - iPad Version : 5060
    QQ HD : 5058
    Followers Analysis Tool For Instagram App Free : 4253
    live.ly - live video streaming : 4145
    Houseparty - Group Video Chat : 3991
    SOMA Messenger : 3232
    Monkey : 3060
    Down To Lunch : 2535
    Flinch - Video Chat Staring Contest : 2134
    Highrise - Your Avatar Community : 2011
    LOVOO - Dating Chat : 1985
    PlayStation®Messages : 1918
    BOO! - Video chat camera with filters & stickers : 1805
    Qzone : 1649
    Chatous - Chat with new people : 1609
    Kiwi - Q&A : 1538
    GhostCodes - a discovery app for Snapchat : 1313
    Jodel : 1193
    FireChat : 1037
    Google Duo - simple video calling : 1033
    Fiesta by Tango - Chat & Meet New People : 885
    Google Allo — smart messaging : 862
    Peach — share vividly : 727
    Hey! VINA - Where Women Meet New Friends : 719
    Battlefield™ Companion : 689
    All Devices for WhatsApp - Messenger for iPad : 682
    Chat for Pokemon Go - GoChat : 500
    IAmNaughty – Dating App to Meet New People Online : 463
    Qzone HD : 458
    Zenly - Locate your friends in realtime : 427
    League of Legends Friends : 420
    豆瓣 : 407
    Candid - Speak Your Mind Freely : 398
    知乎 : 397
    Selfeo : 366
    Fake-A-Location Free ™ : 354
    Popcorn Buzz - Free Group Calls : 281
    Fam — Group video calling for iMessage : 279
    QQ International : 274
    Ameba : 269
    SoundCloud Pulse: for creators : 240
    Tantan : 235
    Cougar Dating & Life Style App for Mature Women : 213
    Rawr Messenger - Dab your chat : 180
    WhenToPost: Best Time to Post Photos for Instagram : 158
    Inke—Broadcast an amazing life : 147
    Mustknow - anonymous video Q&A : 53
    CTFxCmoji : 39
    Lobi : 36
    Chain: Collaborate On MyVideo Story/Group Video : 35
    botman - Real time video chat : 7
    BestieBox : 0
    MATCH ON LINE chat : 0
    niconico ch : 0
    LINE BLOG : 0
    bit-tube - Live Stream Video Chat : 0


We find a similar pattern in the `Reference` and `Social Networking` columns. The Bible App, the Dictionary.com apps, and Google Translate alone could make up one of the most popular genres in itself. Same with social networking, without the big networking apps like Facebook, Pinterest, and Skype, Social Networking would not be among the very top when it comes to the most popular genres. 

On the contrary, the `Finance` genre seems to be more evenly distributed in comparison. If a company was to set about creating a free app in the app store, a finance app seems promising. There is a demand for finance apps, and it is not yet dominated by any particular apps in the way other genres are. 

I would also highlight the `Music` genre in our analysis. This category follows the pattern we observed earlier, where it is dominated by a select few apps. Yet, there is still a strong enough market even outside of the mainstream apps due to the sheer popularity of music apps. Smaller music apps such as ringtone makers and karaoke apps see decent amount of success; not nearly successful as Pandora or Spotify, but enough to make a good profit. 

The cells below show the user distribution amongst these two columns:


```python
for app in ios_free:
    if app[-5] == 'Finance':
        print(app[1], ':', app[5]) # print name and number of ratings
```

    Chase Mobile℠ : 233270
    Mint: Personal Finance, Budget, Bills & Money : 232940
    Bank of America - Mobile Banking : 119773
    PayPal - Send and request money safely : 119487
    Credit Karma: Free Credit Scores, Reports & Alerts : 101679
    Capital One Mobile : 56110
    Citi Mobile® : 48822
    Wells Fargo Mobile : 43064
    Chase Mobile : 34322
    Square Cash - Send Money for Free : 23775
    Capital One for iPad : 21858
    Venmo : 21090
    USAA Mobile : 19946
    TaxCaster – Free tax refund calculator : 17516
    Amex Mobile : 11421
    TurboTax Tax Return App - File 2016 income taxes : 9635
    Bank of America - Mobile Banking for iPad : 7569
    Wells Fargo for iPad : 2207
    Stash Invest: Investing & Financial Education : 1655
    Digit: Save Money Without Thinking About It : 1506
    IRS2Go : 1329
    Capital One CreditWise - Credit score and report : 1019
    U by BB&T : 790
    Paribus - Rebates When Prices Drop : 768
    KeyBank Mobile : 623
    VyStar Mobile Banking for iPhone : 434
    Sparkasse - Your mobile branch : 77
    VyStar Mobile Banking for iPad : 57
    Zaim : 44
    Ma Banque : 17
    Lloyds Bank Mobile Banking : 17
    Suica : 10
    Halifax Mobile Banking : 8
    La Banque Postale : 8
    币优铺 : 0
    Impots.gouv : 0



```python
for app in ios_free:
    if app[-5] == 'Music':
        print(app[1], ':', app[5]) # print name and number of ratings
```

    Pandora - Music & Radio : 1126879
    Spotify Music : 878563
    Shazam - Discover music, artists, videos & lyrics : 402925
    iHeartRadio – Free Music & Radio Stations : 293228
    SoundCloud - Music & Audio : 135744
    Magic Piano by Smule : 131695
    Smule Sing! : 119316
    TuneIn Radio - MLB NBA Audiobooks Podcasts Music : 110420
    Amazon Music : 106235
    SoundHound Song Search & Music Player : 82602
    Sonos Controller : 48905
    Bandsintown Concerts : 30845
    Karaoke - Sing Karaoke, Unlimited Songs! : 28606
    My Mixtapez Music : 26286
    Sing Karaoke Songs Unlimited with StarMaker : 26227
    Ringtones for iPhone & Ringtone Maker : 25403
    Musi - Unlimited Music For YouTube : 25193
    AutoRap by Smule : 18202
    Spinrilla - Mixtapes For Free : 15053
    Napster - Top Music & Radio : 14268
    edjing Mix:DJ turntable to remix and scratch music : 13580
    Free Music - MP3 Streamer & Playlist Manager Pro : 13443
    Free Piano app by Yokee : 13016
    Google Play Music : 10118
    Certified Mixtapes - Hip Hop Albums & Mixtapes : 9975
    TIDAL : 7398
    YouTube Music : 7109
    Nicki Minaj: The Empire : 5196
    Sounds app - Music And Friends : 5126
    SongFlip - Free Music Streamer : 5004
    Simple Radio - Live AM & FM Radio Stations : 4787
    Deezer - Listen to your Favorite Music & Playlists : 4677
    Ringtones for iPhone with Ringtone Maker : 4013
    Bose SoundTouch : 3687
    Amazon Alexa : 3018
    DatPiff : 2815
    Trebel Music - Unlimited Music Downloader : 2570
    Free Music Play - Mp3 Streamer & Player : 2496
    Acapella from PicPlayPost : 2487
    Coach Guitar - Lessons & Easy Tabs For Beginners : 2416
    Musicloud - MP3 and FLAC Music Player for Cloud Platforms. : 2211
    Piano - Play Keyboard Music Games with Magic Tiles : 1636
    Boom: Best Equalizer & Magical Surround Sound : 1375
    Music Freedom - Unlimited Free MP3 Music Streaming : 1246
    AmpMe - A Portable Social Party Music Speaker : 1047
    Medly - Music Maker : 933
    Bose Connect : 915
    Music Memos : 909
    UE BOOM : 612
    LiveMixtapes : 555
    NOISE : 355
    MP3 Music Player & Streamer for Clouds : 329
    Musical Video Maker - Create Music clips lip sync : 320
    Cloud Music Player - Downloader & Playlist Manager : 319
    Remixlive - Remix loops with pads : 288
    QQ音乐HD : 224
    Blocs Wave - Make & Record Music : 158
    PlayGround • Music At Your Fingertips : 150
    Music and Chill : 135
    The Singing Machine Mobile Karaoke App : 130
    radio.de - Der Radioplayer : 64
    Free Music -  Player & Streamer  for Dropbox, OneDrive & Google Drive : 46
    NRJ Radio : 38
    Smart Music: Streaming Videos and Radio : 17
    BOSS Tuner : 13
    PetitLyrics : 0


We have analyzed the popularity of apps in the App Store, now let's analyze the Google Play market. 

### Most Popular Apps by Genre (Google Play) ###

We begin by generating a frequency table for the Google Play app categories. The frequency table will show us the number of installs for each category


```python
categories_android = freq_table(android_free, 1)

avg_installs_by_category = []

for category in categories_android:
    total = 0
    len_category = 0
    for app in android_free:
        category_app = app[1]
        if category_app == category:
            n_installs = app[5]
            n_installs = n_installs.replace('+', '')
            n_installs = n_installs.replace(',', '')
            total += float(n_installs)
            len_category += 1
    if len_category > 0:
        avg_n_installs = total / len_category
        avg_installs_by_category.append((avg_n_installs, category))
        
avg_installs_by_category.sort(reverse=True)

for avg_installs, category in avg_installs_by_category:
    print(category, ':', avg_installs)

```

    COMMUNICATION : 38456119.167247385
    VIDEO_PLAYERS : 24727872.452830188
    SOCIAL : 23253652.127118643
    PHOTOGRAPHY : 17840110.40229885
    PRODUCTIVITY : 16787331.344927534
    GAME : 15588015.603248259
    TRAVEL_AND_LOCAL : 13984077.710144928
    ENTERTAINMENT : 11640705.88235294
    TOOLS : 10801391.298666667
    NEWS_AND_MAGAZINES : 9549178.467741935
    BOOKS_AND_REFERENCE : 8767811.894736841
    SHOPPING : 7036877.311557789
    PERSONALIZATION : 5201482.6122448975
    WEATHER : 5074486.197183099
    HEALTH_AND_FITNESS : 4188821.9853479853
    MAPS_AND_NAVIGATION : 4056941.7741935486
    FAMILY : 3695641.8198090694
    SPORTS : 3638640.1428571427
    ART_AND_DESIGN : 1986335.0877192982
    FOOD_AND_DRINK : 1924897.7363636363
    EDUCATION : 1833495.145631068
    BUSINESS : 1712290.1474201474
    LIFESTYLE : 1437816.2687861272
    FINANCE : 1387692.475609756
    HOUSE_AND_HOME : 1331540.5616438356
    DATING : 854028.8303030303
    COMICS : 817657.2727272727
    AUTO_AND_VEHICLES : 647317.8170731707
    LIBRARIES_AND_DEMO : 638503.734939759
    PARENTING : 542603.6206896552
    BEAUTY : 513151.88679245283
    EVENTS : 253542.22222222222
    MEDICAL : 120550.61980830671


**Google Play Analysis**: 

Based on this data, there is a clear top 3 in the Google Play store: `communication`, `video players`, and `social media`. However, once again these are genres whose data are skewed by a few apps that dominate the market. For example, the communication category has several apps that have over 100 million and 500 million installs:


```python
for app in android_free:
    if app[1] == 'COMMUNICATION' and (app[5] == '1,000,000,000+'
                                      or app[5] == '500,000,000+'
                                      or app[5] == '100,000,000+'):
        print(app[0], ':', app[5])
```

    WhatsApp Messenger : 1,000,000,000+
    imo beta free calls and text : 100,000,000+
    Android Messages : 100,000,000+
    Google Duo - High Quality Video Calls : 500,000,000+
    Messenger – Text and Video Chat for Free : 1,000,000,000+
    imo free video calls and chat : 500,000,000+
    Skype - free IM & video calls : 1,000,000,000+
    Who : 100,000,000+
    GO SMS Pro - Messenger, Free Themes, Emoji : 100,000,000+
    LINE: Free Calls & Messages : 500,000,000+
    Google Chrome: Fast & Secure : 1,000,000,000+
    Firefox Browser fast & private : 100,000,000+
    UC Browser - Fast Download Private & Secure : 500,000,000+
    Gmail : 1,000,000,000+
    Hangouts : 1,000,000,000+
    Messenger Lite: Free Calls & Messages : 100,000,000+
    Kik : 100,000,000+
    KakaoTalk: Free Calls & Text : 100,000,000+
    Opera Mini - fast web browser : 100,000,000+
    Opera Browser: Fast and Secure : 100,000,000+
    Telegram : 100,000,000+
    Truecaller: Caller ID, SMS spam blocking & Dialer : 100,000,000+
    UC Browser Mini -Tiny Fast Private & Secure : 100,000,000+
    Viber Messenger : 500,000,000+
    WeChat : 100,000,000+
    Yahoo Mail – Stay Organized : 100,000,000+
    BBM - Free Calls & Messages : 100,000,000+


If we removed the top communication apps (WhatsApp, Messenger, Skype, etc.), their average number of installs would be reduced significantly. If we were to create an app in any of the communication, video players, and social media categories,it would be nearly impossible to compete against these apps that already have established an oligopoly. 

A game app seems to be a safe recommendation. No matter the platform, games are always at the top of the market. 


```python
for app in android_free:
    if app[1] == 'GAME':
        print(app[0], ':', app[5])
```

    Solitaire : 10,000,000+
    Sonic Dash : 100,000,000+
    PAC-MAN : 100,000,000+
    Bubble Witch 3 Saga : 50,000,000+
    Race the Traffic Moto : 10,000,000+
    Marble - Temple Quest : 10,000,000+
    Shooting King : 10,000,000+
    Geometry Dash World : 10,000,000+
    Jungle Marble Blast : 5,000,000+
    Roll the Ball® - slide puzzle : 100,000,000+
    Block Craft 3D: Building Simulator Games For Free : 50,000,000+
    Farm Fruit Pop: Party Time : 1,000,000+
    Love Balls : 50,000,000+
    Piano Tiles 2™ : 100,000,000+
    Pokémon GO : 100,000,000+
    Paint Hit : 10,000,000+
    Snake VS Block : 50,000,000+
    Rolly Vortex : 10,000,000+
    Woody Puzzle : 1,000,000+
    Stack Jump : 10,000,000+
    The Cube : 5,000,000+
    Extreme Car Driving Simulator : 100,000,000+
    Bricks n Balls : 1,000,000+
    The Fish Master! : 1,000,000+
    Color Road : 10,000,000+
    Draw In : 10,000,000+
    PLANK! : 500,000+
    Looper! : 1,000,000+
    Trivia Crack : 100,000,000+
    Will it Crush? : 5,000,000+
    Tomb of the Mask : 5,000,000+
    Baseball Boy! : 10,000,000+
    Hello Stars : 10,000,000+
    Tank Stars : 10,000,000+
    Hole.io : 10,000,000+
    Mini Golf King - Multiplayer Game : 5,000,000+
    Flip the Gun - Simulator Game : 10,000,000+
    Mad Skills BMX 2 : 1,000,000+
    MMX Hill Dash 2 – Offroad Truck, Car & Bike Racing : 1,000,000+
    Word Link : 10,000,000+
    Last Day on Earth: Survival : 10,000,000+
    Partymasters - Fun Idle Game : 10,000,000+
    Harry Potter: Hogwarts Mystery : 10,000,000+
    Offroad Outlaws : 1,000,000+
    Grim Soul: Dark Fantasy Survival : 5,000,000+
    Disney Heroes: Battle Mode : 5,000,000+
    New YAHTZEE® With Buddies Dice Game : 5,000,000+
    Word Crossy - A crossword game : 5,000,000+
    MLB TAP SPORTS BASEBALL 2018 : 1,000,000+
    Hero Hunters : 5,000,000+
    Cooking Madness - A Chef's Restaurant Games : 10,000,000+
    Ice Crush 2018 - A new Puzzle Matching Adventure : 1,000,000+
    TerraGenesis - Space Colony : 1,000,000+
    SHADOWGUN LEGENDS : 1,000,000+
    RULES OF SURVIVAL : 10,000,000+
    Mafia City : 10,000,000+
    Cash, Inc. Money Clicker Game & Business Adventure : 10,000,000+
    Dino War: Rise of Beasts : 1,000,000+
    Tokyo Ghoul: Dark War : 1,000,000+
    Last Shelter: Survival : 5,000,000+
    Guns of Glory : 10,000,000+
    Lineage 2: Revolution : 5,000,000+
    Final Fantasy XV: A New Empire : 10,000,000+
    Girls' Frontline : 100,000+
    Chapters: Interactive Stories : 1,000,000+
    Honkai Impact 3rd : 1,000,000+
    Master of Eternity(MOE) : 100,000+
    Angry Birds 2 : 100,000,000+
    PUBG MOBILE : 50,000,000+
    The Sims™ FreePlay : 10,000,000+
    Best Fiends - Free Puzzle Game : 10,000,000+
    Jurassic World™ Alive : 5,000,000+
    Choices: Stories You Play : 10,000,000+
    Solitaire TriPeaks : 10,000,000+
    Merge Dragons! : 5,000,000+
    The Walking Dead: Road to Survival : 10,000,000+
    Hustle Castle: Fantasy Kingdom : 10,000,000+
    Might & Magic: Elemental Guardians : 500,000+
    Game of Thrones: Conquest™ : 1,000,000+
    Love Nikki-Dress UP Queen : 5,000,000+
    Summoners War : 50,000,000+
    FINAL FANTASY BRAVE EXVIUS : 5,000,000+
    Fate/Grand Order (English) : 500,000+
    Lords Mobile: Battle of the Empires - Strategy RPG : 50,000,000+
    War and Order : 5,000,000+
    Candy Crush Saga : 500,000,000+
    8 Ball Pool : 100,000,000+
    Subway Surfers : 1,000,000,000+
    Candy Crush Soda Saga : 100,000,000+
    Toon Blast : 10,000,000+
    Toy Blast : 50,000,000+
    Clash Royale : 100,000,000+
    Clash of Clans : 100,000,000+
    Plants vs. Zombies FREE : 100,000,000+
    Block Puzzle : 5,000,000+
    Super Jim Jump - pixel 3d : 1,000,000+
    Pou : 500,000,000+
    Flow Free : 100,000,000+
    Homescapes : 10,000,000+
    Wordscapes : 10,000,000+
    My Talking Angela : 100,000,000+
    slither.io : 100,000,000+
    Cooking Fever : 100,000,000+
    Yes day : 100,000,000+
    Gardenscapes : 50,000,000+
    Fishdom : 10,000,000+
    Galaxy Attack: Alien Shooter : 10,000,000+
    Score! Hero : 100,000,000+
    Magic Tiles 3 : 50,000,000+
    Jewels Star: OZ adventure : 1,000,000+
    Granny : 50,000,000+
    Dream League Soccer 2018 : 100,000,000+
    Sweet Fruit Candy : 10,000,000+
    Fruits Bomb : 10,000,000+
    My Talking Tom : 500,000,000+
    Sniper 3D Gun Shooter: Free Shooting Games - FPS : 100,000,000+
    Shoot Bubble - Fruit Splash : 5,000,000+
    Rider : 10,000,000+
    Zombie Tsunami : 100,000,000+
    Doodle Jump : 50,000,000+
    Helix Jump : 100,000,000+
    Crossy Road : 100,000,000+
    Temple Run 2 : 500,000,000+
    Talking Tom Gold Run : 100,000,000+
    Alto's Adventure : 10,000,000+
    DEER HUNTER 2018 : 10,000,000+
    Cover Fire: offline shooting games for free : 10,000,000+
    Anger of stick 5 : zombie : 50,000,000+
    CATS: Crash Arena Turbo Stars : 50,000,000+
    Soul Knight : 10,000,000+
    Major Mayhem : 10,000,000+
    DINO HUNTER: DEADLY SHORES : 10,000,000+
    Extreme Match : 100,000+
    Power Rangers Dino Charge : 10,000,000+
    Zombie Hunter: Post Apocalypse Survival Games : 10,000,000+
    Agar.io : 100,000,000+
    Zombie Hunter King : 1,000,000+
    Battlelands Royale : 1,000,000+
    Once Upon a Tower : 1,000,000+
    TEKKEN™ : 5,000,000+
    diep.io : 10,000,000+
    Arrow.io : 10,000,000+
    BEYBLADE BURST app : 10,000,000+
    Dragon Hills : 10,000,000+
    Bus Rush: Subway Edition : 100,000,000+
    Tiny Archers : 5,000,000+
    Runner 3d like : 100,000+
    War Robots : 50,000,000+
    Dinosaur Simulator: Dino World : 5,000,000+
    GUNSHIP BATTLE: Helicopter 3D : 50,000,000+
    Quiz: Logo game : 10,000,000+
    Jewels Legend - Match 3 Puzzle : 10,000,000+
    Bubble Shooter : 10,000,000+
    Bubble Shooter Genies : 5,000,000+
    Blossom Blast Saga : 10,000,000+
    Traffic Racer : 100,000,000+
    Hill Climb Racing : 100,000,000+
    Earn to Die 2 : 50,000,000+
    Bubble Shooter 2 : 5,000,000+
    Candy Bomb : 10,000,000+
    Zombie Catchers : 10,000,000+
    Gems or jewels ? : 10,000,000+
    Angry Birds Rio : 100,000,000+
    Candy Crush Jelly Saga : 50,000,000+
    Cut the Rope FULL FREE : 100,000,000+
    Hungry Shark Evolution : 100,000,000+
    Power Pop Bubbles : 10,000,000+
    Angry Birds Classic : 100,000,000+
    Hill Climb Racing 2 : 100,000,000+
    Jewels classic Prince : 5,000,000+
    Fruit Block - Puzzle Legend : 10,000,000+
    Bubble Shooter Space : 1,000,000+
    Swamp Attack : 50,000,000+
    1LINE – One Line with One Touch : 10,000,000+
    Stick War: Legacy : 10,000,000+
    Bowmasters : 50,000,000+
    Block Puzzle Classic Legend ! : 5,000,000+
    Marble Woka Woka 2018 - Bubble Shooter Match 3 : 10,000,000+
    Pixel Art: Color by Number Game : 10,000,000+
    Super Jabber Jump 3 : 5,000,000+
    Rush : 10,000,000+
    Jetpack Joyride : 100,000,000+
    DEAD TARGET: FPS Zombie Apocalypse Survival Games : 50,000,000+
    Word Search : 50,000,000+
    Rope Hero Return of a Legend : 1,000,000+
    Draw a Stickman: EPIC 2 Free : 10,000,000+
    UNO ™ & Friends : 50,000,000+
    4 in a Row : 500,000+
    Super Mario Run : 100,000,000+
    STARDOM: THE A-LIST : 1,000,000+
    Glow Hockey : 100,000,000+
    Asphalt 8: Airborne : 100,000,000+
    Lep's World 2 🍀🍀 : 100,000,000+
    Law of Creation: A Playable Manga : 500,000+
    Real City Car Driver : 10,000,000+
    Rolling Sky : 50,000,000+
    4 in a row : 5,000,000+
    Four In A Line Free : 5,000,000+
    Draw A Stickman : 1,000,000+
    Four In A Line : 1,000,000+
    Cardi B Piano Game : 10,000+
    MORTAL KOMBAT X : 10,000,000+
    ► MultiCraft ― Free Miner! 👍 : 50,000,000+
    Pixel Gun 3D: Survival shooter & Battle Royale : 50,000,000+
    Perfect Piano : 50,000,000+
    Critical Ops : 10,000,000+
    FRONTLINE COMMANDO: D-DAY : 10,000,000+
    Fruit Ninja® : 100,000,000+
    Beach Buggy Blitz : 50,000,000+
    Vector : 100,000,000+
    Dr. Driving : 100,000,000+
    D Day World War II Commando Survival Shooting : 100,000+
    Geometry Dash Meltdown : 50,000,000+
    Bike Race Free - Top Motorcycle Racing Games : 100,000,000+
    Smash Hit : 100,000,000+
    Temple Run : 100,000,000+
    Super Car F. Mod for MCPE : 50,000+
    Gangstar Vegas - mafia game : 50,000,000+
    Rolling G Sky : 100,000+
    G-Switch 2 : 1,000,000+
    G-Switch 3 : 100,000+
    G-Switch : 100,000+
    Radial-G : Infinity : 50,000+
    Geometry Dash Lite : 100,000,000+
    Car Driving Simulator Drift : 1,000,000+
    RC City Police Heavy Traffic Racer : 1,000,000+
    Motorbike Driving Simulator 3D : 10,000,000+
    Helix : 500,000+
    Solitaire! : 10,000,000+
    iGun Pro -The Original Gun App : 10,000,000+
    NinJump : 10,000,000+
    Fidget Spinner : 10,000,000+
    J Balvin Piano Tiles : 500+
    Guess the song of J Balvin : 1,000+
    J-Stars Victory VS Guide : 10,000+
    KIM KARDASHIAN: HOLLYWOOD : 10,000,000+
    Magic Tiles - TWICE Edition (K-Pop) : 100,000+
    Magic Tiles - Blackpink Edition (K-Pop) : 100,000+
    Korean Dungeon: K-Word 1000 : 10,000+
    Lep's World 🍀 : 50,000,000+
    Ant Smasher by Best Cool & Fun Games : 100,000,000+
    Cheating Tom 3 - Genius School : 10,000,000+
    M3-X5-X6-M-İ3-İ8 RACİNG 2018 : 100,000+
    M-acceleration 3D Car Racing : 100,000+
    Super Hero M Craft Run : 100,000+
    Don't Hug Me I'm So Scared : 50,000+
    Rope'n'Fly 4 : 10,000,000+
    Cops N Robbers - FPS Mini Game : 5,000,000+
    Cops N Robbers 2 : 1,000,000+
    Hide N Seek : Mini Game : 500,000+
    Rope'n'Fly 3 - Dusk Till Dawn : 10,000,000+
    Drag'n'Boom : 1,000,000+
    Cops N Robbers : 1,000,000+
    Mopar Drag N Brag : 1,000,000+
    Draw N Guess Multiplayer : 1,000,000+
    Rock N' Cash Casino Slots -Free Vegas Slot Games : 100,000+
    Rock n Roll Music Quiz Game : 10,000+
    Drink-O-Tron The Drinking Game : 50,000+
    The Maze Runner : 10,000,000+
    The Q - Live Trivia Game Network : 100,000+
    Q*bert: Rebooted : 1,000,000+
    Gun Shoot War Q : 1,000,000+
    DJMAX TECHNIKA Q - Music Game : 100,000+
    Stupid Zombies : 10,000,000+
    Crime Wars S. Andreas : 500,000+
    Dino T-Rex : 10,000,000+
    U-48 Submarine Commander Free : 500,000+
    Identity V : 1,000,000+
    V for Voodoo : 1,000,000+
    Sin City Crime Simulator V - Gangster : 100,000+
    The Gang Sniper V. Pocket Edition. : 500,000+
    San Andreas: Gang Crime V : 100,000+
    Grand Stickman Cover V : 100,000+
    Words With Friends Classic : 50,000,000+
    Draw Something Classic : 50,000,000+
    Extreme X Ray Robot Stunts : 5,000,000+
    Casino X - Free Online Slots : 1,000,000+
    X-WOLF : 100,000+
    Hero Fighter X : 500,000+
    Slendrina X : 1,000,000+
    Robocar X Ray : 1,000,000+
    Space X: Sky Wars of Air Force : 1,000,000+
    Real Gangster Crime : 10,000,000+
    Gangster Town : 5,000,000+
    Rope Hero 3 : 5,000,000+
    Experiment Z - Zombie : 10,000,000+
    Z Champions : 1,000,000+
    WithstandZ - Zombie Survival! : 1,000,000+
    Zombie War Z : Hero Survival Rules : 100,000+
    DRAGON BALL Z DOKKAN BATTLE : 10,000,000+
    Cube Z (Pixel Zombies) : 1,000,000+
    GALAK-Z: Variant Mobile : 10,000+
    Reload: The Z-Team : 100,000+
    Rage Z: Multiplayer Zombie FPS Online Shooter : 500,000+
    Pixel Z Gunner 3D - Battle Survival Fps : 1,000,000+
    Saiyan Z: Super SSJ Ultimate Combat : 10,000+
    Mini DAYZ: Zombie Survival : 1,000,000+
    Route Z : 500,000+
    Dragon X Adventure: Warrior Z : 100,000+
    Ultimate Fighter Z : 5,000+
    Angry Birds Friends : 50,000,000+
    Angry Birds Go! : 50,000,000+
    Angry Birds Star Wars : 100,000,000+
    Solitaire: Decked Out Ad Free : 500,000+
    AE Jewels 2: Island Adventures : 10,000+
    AE Bingo: Offline Bingo Games : 500,000+
    AE Jewels : 50,000+
    AE Spider Solitaire : 500,000+
    AE Coin Mania : Arcade Fun : 1,000,000+
    Mupen64+AE FREE (N64 Emulator) : 1,000,000+
    AE Lucky Fishing : 1,000,000+
    AE Video Poker : 50,000+
    AE Solitaire : 50,000+
    AE Blackjack : 100,000+
    AE Gun Ball: arcade ball games : 1,000,000+
    AE 3D MOTOR :Racing Games Free : 5,000,000+
    AE GTO Racing : 100,000+
    AE 3D Moto 3 : 100,000+
    AE FreeCell : 10,000+
    AE City Jump : 10,000+
    AE Angry Chef : 10,000+
    AE Master Moto : 1,000,000+
    AE Fishing Hunter : 1,000+
    Gun Strike Shoot : 10,000,000+
    AG Drive 3D : 10,000+
    VR AG Racing for Cardboard : 1,000+
    AH-1 Viper Cobra Ops - helicopter flight simulator : 100,000+
    Ah! Coins : 50,000+
    Baby Game Animal Jam Free : 500+
    Blackjack aj Poker : 1,000+
    Puppy Shooting an AK-47: Platformer Zombie Game : 1,000+
    Crisis Action: 2018 NO.1 FPS : 10,000,000+
    AK-47 3D : 10,000+
    AK-47 Assult Rifle: Gun Shooting Simulator Game : 500+
    Counter Terrorist Attack : 10,000,000+
    AK Blackjack : 1,000+
    GUN ZOMBIE : 5,000,000+
    Shoot Strike War Fire : 10,000,000+
    I Am Innocent : 1,000,000+
    What Am I? - Brain Teasers : 100,000+
    Who am I? (Biblical) : 500,000+
    Vikings: an Archer's Journey : 1,000,000+
    Draw a Stickman: EPIC Free : 10,000,000+
    THE KING OF FIGHTERS-A 2012(F) : 5,000,000+
    Fast like a Fox : 5,000,000+
    Ao Oni2 : 1,000,000+
    Arena of Valor: 5v5 Arena Game : 10,000,000+
    Mobile Legends: Bang Bang : 100,000,000+
    Kill Shot Bravo: Sniper FPS : 10,000,000+
    Carnivores: Dinosaur Hunter : 1,000,000+
    Hopeless Land: Fight for Survival : 5,000,000+
    Heroes Arena : 10,000,000+
    Undead Assault : 100,000+
    Guess the Class 🔥 AQW : 1,000+
    Iron Blade : 10,000,000+
    Evil Apples: A Dirty Card Game : 5,000,000+
    Ghost Snap AR Horror Survival : 100,000+
    The Walking Dead: Our World : 1,000,000+
    Ingress : 10,000,000+
    Flat Pack : 100,000+
    Crayola Color Blaster : 1,000+
    Guns of Boom - Online Shooter : 10,000,000+
    Guns Royale - Multiplayer Blocky Battle Royale : 100,000+
    Stack It AR : 100,000+
    Arcraft - AR Sandbox : 10,000+
    AR Tank Battle : 10,000+
    Sonic the Hedgehog™ Classic : 10,000,000+
    Pacific Navy Fighter C.E. (AS) : 500,000+
    My OldBoy! Free - GBC Emulator : 5,000,000+
    My Boy! Free - GBA Emulator : 10,000,000+
    SAMURAI vs ZOMBIES DEFENSE : 5,000,000+
    Five Nights at Freddy's 4 Demo : 10,000,000+
    Five Nights at Freddy's 3 Demo : 10,000,000+
    Nights at Cube Pizzeria 3D – 2 : 5,000,000+
    Rivals at War: Firefight : 5,000,000+
    7 Nights at Pixel Pizzeria - 2 : 1,000,000+
    Nights at Slender Pizzeria 3D : 100,000+
    One Night at Golden Freddy's : 100,000+
    Nights at Cube Pizzeria 3D – 4 : 500,000+
    Five Nights at Neighbor House : 100,000+
    Asylum Night Shift - Five Nights Survival : 5,000,000+
    Two Nights at jumpscare : 100,000+
    Five Nights at Pizzeria : 100,000+
    Block Battles: Heroes at War - Multiplayer PVP : 500,000+
    Five Nights at Flappy's : 10,000+
    5 Nights at Cube Pizzeria City : 100,000+
    Au Mobile: Audition Chính Hiệu : 1,000,000+
    AU Mobile Indonesia : 1,000,000+
    Super Dancer : 1,000,000+
    Au-allstar for KR : 100,000+
    Super Dancer VN : 500,000+
    Love Dance : 1,000,000+
    Avatar Musik : 1,000,000+
    Armored Warfare: Assault : 100,000+
    Auction Wars : Storage King : 1,000,000+
    iSniper 3D Arctic Warfare : 1,000,000+
    Modern Action Commando 3D : 100,000+
    Call of Duty®: Heroes : 10,000,000+
    AXE.IO - Brutal Survival Battleground : 500,000+
    Axe Champ : 10,000+
    Axe Champ Hit : 100+
    Axe Champion : 5,000+
    Vikings Stickman Axe Fighting : 100,000+
    Axetatic - Axe Throwing Game : 1,000+
    Axe Champs! Wars : 50+
    Axe Knight : 5,000+
    The Vikings : 1,000,000+
    Knife&Axe Throwing : 50,000+
    Stickman and Axe : 50,000+
    Axe Throw: Flip Champ Axes Champion Throwing Games : 50+
    Axe Man : 1,000+
    Dead Zombie Evil Killer:Axe : 10,000+
    Zlax.io Zombs Luv Ax : 500,000+
    Clash of Axe: Flippy Lumberjack Action X : 1,000+
    Flippy Axe : Flip The Knife & Axe Simulator : 100+
    Cyborg AX-001 : 50+
    Woodman Deluxe : 5,000+
    ¡Ay Metro! : 10,000+
    Ay Vamos - PJ. Balvin - Piano : 5+
    San Andreas Crime Stories : 10,000,000+
    Miami crime simulator : 10,000,000+
    Strange Hero: Future Battle : 10,000,000+
    Mad Day - Truck Distance Game : 5,000,000+
    AZ Rockets : 10,000+
    Ba Cristi : 10,000+
    Banana Kong : 100,000,000+
    Space Shooter : Galaxy Attack : 10,000,000+
    BC Slots - The Lost Reels FREE : 10,000+
    Millionaire : Who want to be? : 100,000+
    Beauty Idol: Fashion Queen : 1,000,000+
    Hard Time (Prison Sim) : 10,000,000+
    Battlefield™ Companion : 10,000,000+
    BF at Sea Refueled : 10,000+
    Bullet Force : 10,000,000+
    BF Combat: Genesis : 100,000+
    Combat BF: Black Ops 3 : 500,000+
    BF Frontline City : 1,000,000+
    Faketalk - Chatbot : 1,000,000+
    Block Gun 3D: Call of Destiny : 1,000,000+
    BLOOD & GLORY: IMMORTALS : 1,000,000+
    Block Gun 3D: Ghost Ops : 5,000,000+
    Rento - Dice Board Game Online : 10,000,000+
    Survival Run with Bear Grylls : 5,000,000+
    VIP Backgammon Free : Play Backgammon Online : 10,000+
    Out of Bounds BH Demo : 10+
    Bi-Tank : 50+
    Bike Rivals : 10,000,000+
    Bingo Party - Free Bingo Games : 1,000,000+
    BJ's Bingo & Gaming Casino : 10,000+
    BJ Bridge Free (2018) : 10,000+
    BJ Bridge Acol Beginner 2018 : 5,000+
    BJ Bridge Standard American 2018 : 1,000+
    BJ Strategy Tester : 100+
    Cards Casino:Video Poker & BJ : 10,000+
    Real Casino:Slot,Keno,BJ,Poker : 50,000+
    Basic Strategy Training BJ 21 : 500+
    BJ card game blackjack : 500+
    BLACKJACK! : 10,000,000+
    Blackjack : 1,000,000+
    BlackJack -21 Casino Card Game : 100,000+
    BlackJack 21 Pro : 500,000+
    Ultimate BlackJack 3D FREE : 100,000+
    Blood Domination - BL Game : 10,000+
    Heirs & Graces : 5,000+
    Lust in Terror Manor - The Truth Unveiled | Otome : 10,000+
    BL!TZ - Endless : 100+
    BL-ED: From BLUE to RED : 10+
    Teen Patti by BL Games : 100,000+
    Word Hunt : 500+
    Skip-Bo™ Free : 1,000,000+
    Red Ball 4 : 50,000,000+
    Skater Boy : 100,000,000+
    Sic Bo Online! Free Casino : 100,000+
    Hambo : 5,000,000+
    Sic Bo Rave : 10,000+
    Thai Sic Bo : 1,000,000+
    Bounce Classic : 10,000,000+
    Bomber Friends : 10,000,000+
    Sic Bo (Tai Xiu) - Multiplayer Casino : 10,000+
    Hungry Shark World : 50,000,000+
    Carros Rebaixados BR : 1,000,000+
    Texas HoldEm Poker Deluxe (BR) : 10,000+
    Brick Breaker BR : 5+
    Black Commando | Special Ops | FPS Shooting : 100,000+
    True Skateboarding Ride Skateboard Game Freestyle : 1,000,000+
    Bullshit! (Free) : 10,000+
    Bullshite! : 10,000+
    Block Strike : 10,000,000+
    BombSquad Remote : 1,000,000+
    BS Tractor : 100+
    BS Chopper : 100+
    B@dL!bs Lite : 1,000+
    kick the buddy : 10,000+
    Bu Hangi Ünlü? : 10+
    Bu Nedir ? : 50+
    BU HANGİ ŞARKI ? - 2018 : 100+
    Bu Hangi Oyun ? : 50+
    Bu Hangi Youtuber ? : 1,000+
    Nedir Bu ? : 10+
    Kick the Buddy : 50,000,000+
    Bu Hangi Dizi ? : 1,000+
    Smashy Road: Arena : 5,000,000+
    BW-Go Free : 10,000+
    BW-Joseki : 10,000+
    BW-DGS plugin : 1,000+
    BW-GnuGo : 5,000+
    BW Switch : 10+
    C3-C4-PİCASSO-ELYSEE RACİNG : 1,000+
    Driving Cars Simulator Citroen : 100+
    RC Monster Truck - Offroad Driving Simulator : 5,000,000+
    Super Jabber Jump : 10,000,000+
    Commando Adventure Assassin : 5,000,000+
    Roulette Advisor LITE : 5,000+
    Mini Motor Racing WRT : 1,000,000+
    BMX Boy Bike Stunt Rider Game : 10,000+
    World of Warriors: Duel : 500,000+
    Driving Zone : 1,000,000+
    Color by Number – New Coloring Book : 5,000,000+
    Car Race by Fun Games For Free : 1,000,000+
    Slot Machines by IGG : 10,000,000+
    Bingo by IGG: Top Bingo+Slots! : 5,000,000+
    Voxel - 3D Color by Number & Pixel Coloring Book : 1,000,000+
    BZ Zombie VR : 1,000+
    BZ Fingers : 50+
    Sniper Gun Shot 3D: Impossible Missions : 100+
    Elite Commando Shooting War : 100+
    Animal Hunting: Sniper Shooting : 50+
    Creative Destruction : 5,000,000+
    Sonic CD Classic : 1,000,000+
    ARK: Survival Evolved : 500,000+
    Grim Tales: The Wishes CE : 100,000+
    CrossFire: Legends : 5,000,000+
    Special Forces Group 2 : 10,000,000+
    Motocross Motorbike Simulator Offroad : 5,000,000+
    Sermon on Proverbs CH Spurgeon : 10,000+
    Chicken Invaders 3 : 5,000,000+
    Trovami se ci riesci : 10+
    CJ: Strike Back : 1,000,000+
    The Grand Wars: San Andreas : 1,000,000+
    The Grand Way : 500,000+
    San Andreas City : Auto Theft Car gangster : 100,000+
    Miami Crime Vice Town : 10,000,000+
    CJ Poker Odds Calculator : 50,000+
    Gangster City Auto Theft : 50,000+
    China Town Auto Theft 2 : 50,000+
    Grand Gangsters 3D : 10,000,000+
    Gang Wars of San Andreas : 1,000,000+
    Sin City Hero : Crime Simulator of Vegas : 100,000+
    Zombie Death Shooter : 1,000,000+
    Project Grand Auto Town Sandbox Beta : 500,000+
    CONTRACT KILLER: ZOMBIES (NR) : 5,000,000+
    CKZ ORIGINS : 1,000,000+
    211:CK Lite : 10+
    CONTRACT KILLER: ZOMBIES : 5,000,000+
    Can Knockdown 3 : 10,000,000+
    Sky Streaker - Gumball : 5,000,000+
    Just A Regular Arcade : 1,000,000+
    Dreamland Arcade - Steven Universe : 500,000+
    Adventure Time Run : 1,000,000+
    Glitch Fixers: Powerpuff Girls : 5,000,000+
    StirFry Stunts - We Bare Bears : 10,000,000+
    Angelo Rules - Crazy day : 1,000,000+
    Multicraft Miner Exploration : 1,000,000+
    Alex & Co Quiz : 1,000+
    Them Bombs: co-op board game play with 2-4 friends : 100,000+
    Chinese Chess ( Xiangqi Free ) : 50,000+
    Chinese Chess / Co Tuong : 500,000+
    Call of Mini™ Zombies 2 : 5,000,000+
    2-Player Co-op Zombie Shoot : 1,000+
    IV Go（get IV for Pokemon） : 1,000,000+
    CP Trivia : 100+
    CQ: Lost In Transmission : 10+
    Palavras la cq : 100,000+
    Gunship Modern Combat 3D : 500,000+
    Sniper Killer Shooter : 10,000,000+
    Snowboard Racing Free Fun Game : 100,000+
    Toughest Game Ever 2 : 5,000,000+
    RIDE ZERO : 100,000+
    Critical Strike CS: Counter Terrorist Online FPS : 1,000,000+
    Assault Line CS Online Fps Go : 1,000,000+
    Standoff 2 : 10,000,000+
    Counter Online FPS Game : 5,000,000+
    Combat Strike CS 🔫 Counter Terrorist Attack FPS💣 : 100,000+
    Ultimate Quiz for CS:GO - Skins | Cases | Players : 100,000+
    Аim Training for CS : 100,000+
    Sniper Traning for CS GO : 50,000+
    Your rank CS:GO : 100,000+
    Mobile CS:GO : 100,000+
    Counter Terrorist Gun Strike CS: Special Forces : 10,000+
    Shooter Sniper CS - FPS Games : 1,000,000+
    CS16Client : 500,000+
    Counter Attack - Multiplayer FPS : 1,000,000+
    What's Your CS:GO rank? : 10,000+
    Map Callouts for CS:GO : 50,000+
    Quiz CS:GO players : 50,000+
    Puzzle for CS:GO : 10,000+
    CT-ART 4.0 (Chess Tactics 1200-2400 ELO) : 100,000+
    Moto Rider : 10,000,000+
    Shadow Fight 2 : 100,000,000+
    Crazy Wheels : 10,000,000+
    Cartoon Wars: Blade : 5,000,000+
    Racing CX : 500+
    Freecell CY : 50,000+
    Cytus : 5,000,000+
    Shadowverse CCG : 1,000,000+
    SuperBikers 2 : 500,000+
    Dan the Man: Action Platformer : 10,000,000+
    Geometry Dash SubZero : 10,000,000+
    Metal Soldiers 2 : 10,000,000+
    Quiz TRUE or FALSE : 50,000+
    Run Sausage Run! : 10,000,000+
    Knife Hit : 10,000,000+
    The Visitor : 5,000,000+
    Just Dance Now : 10,000,000+
    DRAGON BALL LEGENDS : 5,000,000+
    DB Ultra Saiyan Battle : 1,000+
    Injustice 2 : 5,000,000+
    Injustice: Gods Among Us : 10,000,000+
    Ultimate DC Challenge : 10,000+
    Justice League Action Run : 1,000,000+
    Quiz DC : 1,000+
    MARVEL Contest of Champions : 50,000,000+
    MARVEL Avengers Academy : 10,000,000+
    DC Universe Quiz : 500+
    Power Rangers: Legacy Wars : 10,000,000+
    WDAMAGE: Car Crash Engine : 1,000,000+
    Word Search multilingual : 1,000,000+
    Piano Free - Keyboard with Magic Tiles Music Games : 50,000,000+
    Racing in Car 2 : 50,000,000+
    Bike Racing 3D : 50,000,000+
    Truck Driver Cargo : 10,000,000+
    Checkers : 10,000,000+
    Scratch Logo Quiz. Challenging brain puzzle : 10,000,000+
    DF Squid : 100+
    DG Mobile : 1,000+
    Destroy Gunners Σ : 1,000,000+
    DH Texas Poker - Texas Hold'em : 10,000,000+
    DEER HUNTER CLASSIC : 50,000,000+
    DEER HUNTER RELOADED : 5,000,000+
    DH Pineapple Poker OFC : 100,000+
    Call of Mini™ Dino Hunter : 10,000,000+
    Bike Mayhem Free : 10,000,000+
    Car Crash III Beam DH Real Damage Simulator 2018 : 10,000+
    DEER HUNTER CHALLENGE : 5,000,000+
    Defender : 10,000,000+
    Texas HoldEm Poker Deluxe : 10,000,000+
    Deck Heroes: Legacy : 10,000,000+
    Shred! Downhill Mountainbiking : 1,000,000+
    Archery Physics Objects Destruction Apple shooter : 100,000+
    Fancy Pants Adventures : 1,000,000+
    Lep's World 3 🍀🍀🍀 : 50,000,000+
    Robbery Bob : 10,000,000+
    Nyan Cat: Lost In Space : 10,000,000+
    DK Quiz : 50,000+
    Super DK vs Kong Brother Advanced Free Classic : 10,000+
    DM EVOLUTION : 10,000+
    DM Adventure : 10+
    Destiny Ninja Shall we date otome games love story : 500,000+
    Free Poker Games : Downtown Casino - Texas Holdem : 50,000+
    NARUTO X BORUTO NINJA VOLTAGE : 5,000,000+
    Does not Commute : 5,000,000+
    Kung Fu Do Fighting : 10,000,000+
    How well do you know me? : 100,000+
    Dude Perfect 2 : 10,000,000+
    Endless Ducker : 500,000+
    that's lit : 100,000+
    Dash Quest Heroes : 500,000+
    Jungle book-The Great Escape : 100,000+
    Bunny Skater : 10,000,000+
    The Walking Zombie: Dead City : 1,000,000+
    Dr. Parking 4 : 50,000,000+
    Dr. Driving 2 : 10,000,000+
    Dr. Parker : Parking Simulator : 1,000,000+
    Dr. Parker : Real car parking simulation : 1,000,000+
    Dr. Rocket : 1,000,000+
    Dr. Dominoes : 500,000+
    Dr. Gomoku : 5,000,000+
    Dr. Chess : 1,000,000+
    Dr Dre - Beatmaker : 10,000+
    Dr Driving Racer : 10,000+
    Dr. Shogi : 1,000,000+
    NDS Emulator - For Android 6 : 1,000,000+
    Free DS Emulator : 1,000,000+
    nds4droid : 10,000,000+
    MegaNDS (NDS Emulator) : 500,000+
    EmuBox - Fast Retro Emulator : 50,000+
    Simple x3DS Emulator - BETA : 50,000+
    DS Tower Defence : 100,000+
    RetroArch : 1,000,000+
    Wheelie Challenge : 5,000,000+
    Sword of Chaos - Lame du Chaos : 100,000+
    Hotel Insanity : 100,000+
    Shoot! DX - lights for FREE : 100+
    American Sniper City Fight Shooting Assassin : 50,000+
    Street Skater 3D : 10,000,000+
    DZ-JOKER : 100+
    Super ball DZ : 1,000+
    Ludo Dz : 500+
    Need for Speed™ No Limits : 50,000,000+
    Peggle Blast : 5,000,000+
    Heroes of Dragon Age : 1,000,000+
    SCRABBLE : 5,000,000+
    Asphalt Xtreme: Rally Racing : 10,000,000+
    Modern Combat 5: eSports FPS : 100,000,000+
    Mass Effect: Andromeda APEX HQ : 100,000+
    Mirror’s Edge™ Companion : 100,000+
    Gear.Club - True Racing : 1,000,000+
    Mental Hospital:EB 2 Lite : 100,000+
    EC Mover : 10+
    EF Jumper : 100+
    E.G. Chess Free : 10,000+
    L.A. Crime Stories 2 Mad City Crime : 100,000+
    LA Stories 4 New Order Sandbox 2018 : 1,000,000+
    L.A. Crime Stories Mad City Crime : 1,000,000+
    Miraculous Ladybug & Cat Noir - The Official Game : 10,000,000+
    Betul Ke Bohong Eh? : 100,000+
    Eh Amego! : 500,000+
    MazeMilitia Classic Multiplayer Shooting Game : 50,000+
    ZOMBIE RIPPER : 500,000+
    Horror Hospital : 1,000,000+
    Mad Skills Motocross : 1,000,000+
    Pineapple Pen : 10,000,000+
    Guess The Emoji : 5,000,000+
    Camping RV Caravan Parking 3D : 500,000+
    Grand Bat Superhero Flying Assault Rescue Mission : 500,000+
    Zombie Frontier : 10,000,000+
    RUN JUMP RUN-fun games for free : 100,000+
    Tamago egg : 5,000+
    Let's Poke The Egg : 1,000,000+
    Survival: Prison Escape : 10,000,000+
    TAMAGO Monsters Returns : 1,000,000+
    Traffic Sniper Counter Attack : 1,000,000+
    Rope Superhero Unlimited : 1,000,000+
    Extreme Super Car Driving 3D : 1,000,000+
    Fast Racing Car Simulator : 1,000,000+
    Beach Head Shooting Assault : 100,000+
    Stickman Warriors Heroes 2 : 1,000,000+
    Dead Target Zombie Shooting US Sniper Killer Squad : 50,000+
    Dubai Racing : 500,000+
    Robot Fighting Games™ - Real Boxing Champions 3D : 5,000+
    Ghost Hunting camera : 500,000+
    Santa Panda Bubble Christmas : 10,000+
    Racing Moto : 50,000,000+
    Train Racing Games 3D 2 Player : 10,000,000+
    Adivina el cantante de Trap y Reggaeton : 100,000+
    Fernanfloo : 10,000,000+
    Adivina el Emoji : 100,000+
    Red Hands – 2-Player Games : 10,000,000+
    Get 'Em : 500,000+
    Live Hold’em Pro Poker - Free Casino Games : 10,000,000+
    Beach Shoot Em Up: Head Hunter : 500,000+
    Stick 'Em Up 2 Starter Edition : 100,000+
    Shoot`Em Down: Shooting game : 500,000+
    Texas Hold’em Poker + | Social : 500,000+
    PlayTexas Hold'em Poker Free : 1,000,000+
    HAWK – Force of an Arcade Shooter. Shoot 'em up : 5,000,000+
    Texas Hold'em Poker : 1,000,000+
    Poker Live! 3D Texas Hold'em : 100,000+
    Fun Texas Hold'em Poker : 500,000+
    Dragonplay™ Poker Texas Holdem : 1,000,000+
    Punch em : 500,000+
    Zynga Poker – Texas Holdem : 50,000,000+
    Boyaa Poker (En) – Social Texas Hold’em : 1,000,000+
    World Series of Poker – WSOP Free Texas Holdem : 10,000,000+
    Texas Holdem Poker Pro : 5,000,000+
    Rage Fight of Streets - Beat Em Up Game : 100,000+
    HANGAME Casino - Baccarat & Texas Hold'em : 500,000+
    PokerStars Play: Free Texas Holdem Poker Game : 500,000+
    Texas Holdem Poker : 10,000,000+
    Big Fish Casino – Play Slots & Vegas Games : 10,000,000+
    TRANSFORMERS: Forged to Fight : 10,000,000+
    Texas Holdem & Omaha Poker: Pokerist : 10,000,000+
    Governor of Poker 2 - OFFLINE POKER GAME : 5,000,000+
    Squadron - Bullet Hell Shooter : 10,000,000+
    Words With Friends – Play Free : 10,000,000+
    Classic Words Solo : 5,000,000+
    Chess Free : 50,000,000+
    Sports Car Driving Simulator 2018 : 500,000+
    The Great Wobo Escape Ep. 1 : 500,000+
    The Visitor: Ep.2 - Sleepover Slaughter : 1,000,000+
    The Visitor: Ep.1 - Kitty Cat Carnage : 500,000+
    Dr.Slender Ep 1 Guide (Eng) : 10,000+
    Decay: The Mare - Ep.1 (Trial) : 100,000+
    EP Gem Hunter : 1,000+
    Gate Of Death Ep: 1 : 100+
    Oggy : 5,000,000+
    Invasion: Defend EU : 50+
    R-EU-READY : 500+
    Speed Racing Ultimate 2 : 1,000,000+
    Countries of the European Union (Quiz) : 100+
    I Know Stuff : 5,000,000+
    Water Surfer Racing In Moto : 1,000,000+
    Ew, the small alien : 10+
    Rescue Robots Survival Games : 5,000,000+
    Bike Race - Bike Blast Rush : 10,000,000+
    ETERNITY WARRIORS 2 : 5,000,000+
    Snes9x EX+ : 5,000,000+
    Virtual Dice EX : 5,000+
    Eyes - The Scary Horror Game Adventure : 10,000,000+
    Sudoku Master : 1,000,000+
    Jungle Monkey Run : 10,000,000+
    Ez Mirror Match : 500,000+
    CNY Slots : Gong Xi Fa Cai 发财机 : 5,000+
    Golden HoYeah Slots - Real Casino Slots : 5,000,000+
    FaFaFa™ Gold Casino: Free slot machines : 1,000,000+
    Heart of Vegas™ Slots – Free Slot Casino Games : 10,000,000+
    Garena Free Fire : 100,000,000+
    FG SPINNER : 10+
    BROTHER IN WARS: GUNNER CITY WARLORDS : 1,000,000+
    Angry Birds Space HD : 5,000,000+
    Police Car Driver : 10,000,000+
    Modern Sniper Strike: Best Commando Action 2k18 : 10,000+
    Moto Fighter 3D : 10,000,000+
    BMX Boy : 50,000,000+
    Crazy Bike attack Racing New: motorcycle racing : 5,000,000+
    Fields of Battle : 1,000,000+
    Family Guy The Quest for Stuff : 10,000,000+
    Toy Attack : 500,000+
    SnowMobile Parking Adventure : 1,000,000+
    Zombie Sniper 3D III : 500,000+
    Shoot Hunter-Gun Killer : 50,000,000+
    Fast Motorcycle Driver 2016 : 1,000,000+
    Forgotten Hill: Fall : 50,000+
    Forgotten Hill Mementoes : 100,000+
    Forgotten Hill: Surgery : 100,000+
    Forgotten Hill: Puppeteer : 100,000+
    FJ Final Join , Circles Game : 1,000+
    Offroad drive : 4x4 driving game : 100,000+
    Rope Hero: Vice Town : 10,000,000+
    Drive 4x4 Luxury SUV Jeep : 500,000+
    Motocross Mayhem : 1,000,000+
    Dino Defends king 3 – Dinosaur T rex Hunter Games : 50,000+
    Block Gun 3D: Haunted Hollow : 1,000,000+
    Navy Gunner Shoot War 3D : 10,000,000+
    Drift Legends : 1,000,000+
    FK (FlightKid) : 10+
    Toy Truck Rally 3D : 50,000,000+
    PRO MX MOTOCROSS 2 : 5,000,000+
    FRONTLINE COMMANDO : 10,000,000+
    Gun Builder ELITE : 1,000,000+
    Magnum 3.0 Gun Custom SImulator : 1,000,000+
    Gun Club Armory : 1,000,000+
    Armed Cam Gun Pack : 10,000+
    Fo Fo Fish : 50+
    4x4 Jeep Racer : 1,000,000+
    Frontline Terrorist Battle Shoot: Free FPS Shooter : 1,000,000+
    Mad Dash Fo' Cash : 100+
    Race Manager FP : 5,000+
    FP Runner : 1,000+
    Modern Counter Terrorist FPS Shoot : 100,000+
    Monster Ride Pro : 10+
    BEBONCOOL GAMEPAD V1.0 : 100,000+
    Modern Strike Online : 10,000,000+
    Modern Counter Terror Attack – Shooting Game : 50,000+
    Big Hunter : 10,000,000+
    Modern Counter Global Strike 3D : 50,000+
    Modern Counter Global Strike 3D V2 : 50,000+
    Winter Wonderland : 50,000+
    Soccer Clubs Logo Quiz : 1,000,000+
    Sid Story : 500,000+
    Fatal Raid - No.1 Mobile FPS : 1,000,000+
    Poker Pro.Fr : 100,000+


Another recommendation would be a health and fitness app. Fitness is something that a lot of people seem to be interested in. There is a ton of variety and small apps that see a lot of sucess. 


```python
for app in android_free:
    if app[1] == 'HEALTH_AND_FITNESS':
        print(app[0], ':', app[5])
```

    Step Counter - Calorie Counter : 500,000+
    Lose Belly Fat in 30 Days - Flat Stomach : 5,000,000+
    Pedometer - Step Counter Free & Calorie Burner : 1,000,000+
    Six Pack in 30 Days - Abs Workout : 10,000,000+
    Lose Weight in 30 Days : 10,000,000+
    Pedometer : 10,000,000+
    LG Health : 10,000,000+
    Step Counter - Pedometer Free & Calorie Counter : 10,000,000+
    Pedometer, Step Counter & Weight Loss Tracker App : 10,000,000+
    Sportractive GPS Running Cycling Distance Tracker : 1,000,000+
    30 Day Fitness Challenge - Workout at Home : 10,000,000+
    Home Workout for Men - Bodybuilding : 1,000,000+
    Fat Burning Workout - Home Weight lose : 100,000+
    Buttocks and Abdomen : 500,000+
    Walking for Weight Loss - Walk Tracker : 100,000+
    Running & Jogging : 500,000+
    Sleep Sounds : 1,000,000+
    Fitbit : 10,000,000+
    Lose Belly Fat-Home Abs Fitness Workout : 50,000+
    Cycling - Bike Tracker : 500,000+
    Abs Training-Burn belly fat : 100,000+
    Calorie Counter - EasyFit free : 1,000,000+
    Aunjai i lert u : 500,000+
    Garmin Connect™ : 10,000,000+
    BetterMe: Weight Loss Workouts : 5,000,000+
    Bike Computer - GPS Cycling Tracker : 1,000,000+
    Six Packs for Man–Body Building with No Equipment : 100,000+
    Running Distance Tracker + : 1,000,000+
    The TK-App - everything under control : 100,000+
    Runkeeper - GPS Track Run Walk : 10,000,000+
    Walking: Pedometer diet : 1,000,000+
    Recipes for hair and face tried : 500,000+
    Abs Workout - 30 Days Fitness App for Six Pack Abs : 100,000+
    8fit Workouts & Meal Planner : 10,000,000+
    Keep Trainer - Workout Trainer & Fitness Coach : 1,000,000+
    Couch to 10K Running Trainer : 500,000+
    PumpUp — Fitness Community : 1,000,000+
    Home workouts - fat burning, abs, legs, arms,chest : 1,000,000+
    Running Weight Loss Walking Jogging Hiking FITAPP : 1,000,000+
    Strava Training: Track Running, Cycling & Swimming : 10,000,000+
    Workout Trainer: fitness coach : 10,000,000+
    Fabulous: Motivate Me! Meditate, Relax, Sleep : 5,000,000+
    StrongLifts 5x5 Workout Gym Log & Personal Trainer : 1,000,000+
    Run with Map My Run : 5,000,000+
    7 Minute Workout : 10,000,000+
    Workout Tracker & Gym Trainer - Fitness Log Book : 500,000+
    Freeletics: Personal Trainer & Fitness Workouts : 10,000,000+
    Fitbit Coach : 1,000,000+
    JEFIT Workout Tracker, Weight Lifting, Gym Log App : 5,000,000+
    Map My Ride GPS Cycling Riding : 1,000,000+
    Daily Workouts - Exercise Fitness Routine Trainer : 10,000,000+
    Google Fit - Fitness Tracking : 10,000,000+
    Weight Loss Running by Verv : 1,000,000+
    Sworkit: Workouts & Fitness Plans : 5,000,000+
    Map My Fitness Workout Trainer : 1,000,000+
    Sports Tracker Running Cycling : 5,000,000+
    Seven - 7 Minute Workout Training Challenge : 1,000,000+
    Daily Yoga - Yoga Fitness Plans : 5,000,000+
    Relax Meditation: Sleep with Sleep Sounds : 1,000,000+
    Free Meditation - Take a Break : 100,000+
    Meditate OM : 1,000,000+
    Yoga - Track Yoga : 500,000+
    My Chakra Meditation : 500,000+
    Relax with Andrew Johnson Lite : 100,000+
    Meditation Music - Relax, Yoga : 1,000,000+
    Down Dog: Great Yoga Anywhere : 500,000+
    21-Day Meditation Experience : 100,000+
    My Chakra Meditation 2 : 100,000+
    Simply Yoga - Fitness Trainer for Workouts & Poses : 1,000,000+
    Relax Melodies: Sleep Sounds : 5,000,000+
    Simple Habit Meditation : 500,000+
    Headspace: Meditation & Mindfulness : 10,000,000+
    Yoga Studio: Mind & Body : 100,000+
    Pregnancy & Baby Tracker : 1,000,000+
    My Cycles Period and Ovulation : 1,000,000+
    I’m Expecting - Pregnancy App : 1,000,000+
    The Bump Pregnancy Tracker : 1,000,000+
    My Days - Ovulation Calendar & Period Tracker ™ : 5,000,000+
    Best Ovulation Tracker Fertility Calendar App Glow : 1,000,000+
    Eve Period Tracker - Love, Sex & Relationships App : 1,000,000+
    Fertility Friend Ovulation App : 1,000,000+
    Pregnancy Tracker : 500,000+
    Period Tracker : 10,000,000+
    Spot On Period, Birth Control, & Cycle Tracker : 500,000+
    OvuView: Ovulation and Fertility : 1,000,000+
    Period Tracker - Period Calendar Ovulation Tracker : 100,000,000+
    Period Tracker Clue: Period and Ovulation Tracker : 10,000,000+
    Calorie Counter - MyFitnessPal : 50,000,000+
    WomanLog Calendar : 5,000,000+
    Geocaching® : 5,000,000+
    Map My Hike GPS Hiking : 500,000+
    ViewRanger - Hike, Ride or Walk : 1,000,000+
    Runtastic Mountain Bike GPS Tracker : 1,000,000+
    AllTrails: Hiking, Running & Mountain Bike Trails : 1,000,000+
    Water Drink Reminder : 10,000,000+
    Zombies, Run! 5k Training (Free) : 50,000+
    Tracks : 50,000+
    YAZIO Calorie Counter, Nutrition Diary & Diet Plan : 5,000,000+
    Technutri - calorie counter, diet and carb tracker : 5,000,000+
    Couch to 5K by RunDouble : 1,000,000+
    Fooducate Healthy Weight Loss & Calorie Counter : 1,000,000+
    Weight Watchers Mobile : 5,000,000+
    Walk with Map My Walk : 5,000,000+
    My Diet Coach - Weight Loss Motivation & Tracker : 10,000,000+
    Endomondo - Running & Walking : 10,000,000+
    Nike+ Run Club : 10,000,000+
    Lose It! - Calorie Counter : 10,000,000+
    Runtastic Running App & Mile Tracker : 10,000,000+
    Calorie Counter - MyNetDiary : 1,000,000+
    10 Best Foods for You : 500,000+
    MyPlate Calorie Tracker : 1,000,000+
    My Diet Diary Calorie Counter : 1,000,000+
    Calorie Counter - Macros : 100,000+
    Weight Loss Tracker - RecStyle : 1,000,000+
    Lark - 24/7 Health Coach : 100,000+
    MealLogger-Photo Food Journal : 50,000+
    Health and Nutrition Guide : 500,000+
    Calorie Counter & Diet Tracker : 1,000,000+
    Food Calorie Calculator : 100,000+
    Monitor Your Weight : 5,000,000+
    Calorie Counter by FatSecret : 10,000,000+
    Eat Fit - Diet and Health Free : 100,000+
    Self Healing : 500,000+
    Happify : 100,000+
    Binaural Beats Therapy : 1,000,000+
    Pacifica - Stress & Anxiety : 500,000+
    Relaxing Sounds : 100,000+
    White Sound Pro : 500,000+
    Insight Timer - Free Meditation App : 1,000,000+
    Self-help Anxiety Management : 500,000+
    Brain Waves - Binaural Beats : 500,000+
    Prana Breath: Calm & Meditate : 1,000,000+
    7 Cups: Anxiety & Stress Chat : 500,000+
    Calm - Meditate, Sleep, Relax : 5,000,000+
    White Noise Lite : 1,000,000+
    Stop, Breathe & Think: Meditation & Mindfulness : 1,000,000+
    Oral-B App : 1,000,000+
    f.lux (preview, root-only) : 500,000+
    HPlus : 100,000+
    H Band 2.0 : 500,000+
    H-Connect : 5,000+
    H Band : 10,000+
    J - Style Pro : 10,000+
    K Health : 50,000+
    Vi Trainer : 5,000+
    30 Day Ab Challenge : 500,000+
    Ultimate Ab & Core Workouts : 1,000,000+
    Abs Workout - Burn Belly Fat with No Equipment : 10,000,000+
    6 Pack Promise - Ultimate Abs : 1,000,000+
    Abs, Core & Back Workout Challenge : 100,000+
    AB Mobile App : 100,000+
    Daily Ab Workout - Core & Abs Fitness Exercises : 10,000,000+
    Abs workout 7 minutes : 1,000,000+
    30 Day Ab Challenge FREE : 1,000,000+
    ABS Workout - Belly workout, 30 days AB : 100,000+
    7 minute abs workout - Daily Ab Workout : 100,000+
    5 Minute Ab Workouts : 50,000+
    30 Day AB Challenge - Lumowell : 10,000+
    Ab Workouts : 1,000,000+
    Ab Workouts - Ab Generator : 1,000+
    Abs workout - 21 Day Fitness Challenge : 1,000,000+
    Ladies' Ab Workout FREE : 1,000,000+
    30-Day Ab Challenge Tracker : 10,000+
    AF Hydro : 10,000+
    AF Nutrition - Integratori : 100+
    AH Connect (Adventist Health) : 1,000+
    be'ah : 10+
    We're Working Out - Al Kavadlo : 50,000+
    AQ Dentals : 10+
    Sleep as Android Gear Addon : 100,000+
    Sleep as Android Garmin Addon : 10,000+
    Samsung Health : 500,000,000+
    Runtastic Sleep Better: Sleep Cycle & Smart Alarm : 5,000,000+
    Twilight: Blue light filter : 5,000,000+
    MediBeat for AW - Android (1) : 500+
    Q7 SmartWatch : 10,000+
    AY Oakmont : 10+
    Cures A-Z : 100,000+
    BA Accesible : 1,000+
    Fitness Dance for Zum.ba Workout Exercise : 50,000+
    BD Provider App : 5,000+
    BF Scale Health Fitness Tool : 1,000+
    Women"s Health Tips(Breast,Face,Body,weight lose) : 1,000,000+
    Revita.bg : 10+
    Blood Glucose Tracker : 100,000+
    BH by Kinomap : 10,000+
    BH - Fitness & Nutrition : 1+
    FitConsole : 50,000+
    Run on Earth : 50,000+
    Treadmill Workouts Free (P) : 1,000+
    The Daily BJ : 500+
    Hungry Girl Diet Bk. Companion : 1,000+
    BL ONLINE PERSONAL TRAINING : 5+
    Poop Tracker - Toilet Log : 10,000+
    BM Pharmacy : 1,000+
    BM Physiotherapy Clinic : 100+
    My BP Lab : 10,000+
    BP Tracker : 1,000+
    MI-BP : 50+
    Deep Breathe BP : 1,000+
    BP Toolbox : 1,000+
    Blood pressure : 500,000+
    Bacterial Vaginosis : 100+
    Bacterial Vaginosis 🇺🇸 : 500+
    Bacteria Vaginosis : 1,000,000+
    Bacterial vaginosis Treatment - Sexual disease : 500+
    Home Remedies for Bacterial Infections : 50+
    Benefit Extras Mobile App : 1,000+
    Kick Axe Bx : 1+
    CB Martial Arts : 10+
    CB NFC Pendant : 50+
    CB Fit : 10+
    C B Patel Health Club : 100+
    CF SPOT : 100+
    CF Talenti : 100+
    Thrive CF : 100+
    CF Townsville : 100+
    CF Etowah : 100+
    Cystic Fibrosis Symptoms, Doctors & Treatments : 100+
    CF Themis : 100+
    CF Riga : 100+
    Casa CF : 500+
    CampGladiator : 50,000+
    CG Fit Scale : 1,000+
    Pharmacy CI - Pharmacies de garde Côte d'Ivoire : 100,000+
    Pharmacie de Garde CI et Prix : 100,000+
    CJ Fitness : 10,000+
    The CJ Rubric : 100+
    CK SKILLZ : 50+
    CK Life : 100+
    CK Pharmacies : 100+
    CK Active : 10+
    CL-Customer Care : 5+
    CL Strength : 50+
    The ClubHouse CR : 100+
    Evolution Marketing CR : 1,000+
    I AM C.T. : 1,000+
    Cy's Elma Pharmacy : 10+
    DG Fitness : 100+
    DG Xplained : 100+
    Stop Smoking - EasyQuit free : 500,000+
    Dr. Muscle : 10,000+
    DS Companion : 50+
    Eat Right Diet (by Dt Shreya's Family Diet Clinic) : 10+
    Dt. Jyothi Srinivas : 100+
    DT NO.I : 1,000+
    Bodyworks DW : 10+
    D.W Bien Être : 10+
    Cloud DX Connected Health : 100+
    DY Fitness : 10+
    PHARMAGUIDE (DZ) : 5,000+
    DEM DZ : 100+
    Eb & flow Yoga Studio : 10+
    EF Coach : 500+
    EF Academy : 50+
    Period Tracker, Pregnancy Calculator & Calendar 🌸 : 10,000+
    EO App. SelfCompassion to you : 100+
    Nike Training Club - Workouts & Fitness Plans : 10,000,000+
    Ideal Weight, BMI Calculator : 5,000,000+
    Fitness & Bodybuilding : 5,000,000+
    CALIOPE EU: Air Quality : 1,000+
    Home Workout - No Equipment : 10,000,000+
    EW MOTION THERAPY : 10+
    SCI-Ex : 1,000+
    X your Ex - Break Up Treatment : 5,000+
    Dream EZ : 10,000+
    FD Fitness : 50+
    L!FE Premium Training : 100+
    Santa Fe Thrive : 50+
    Burn Your Fat With Me! FG : 1,000,000+
    FH Calculator : 500+
    Restaurant Inspections - FL : 10,000+
    Florida Blue : 100,000+


### Final Conclusions ###

In this project, we analyzed data about the most popular apps on the App Store and Google Play. Our goal was to find a genre of apps that we can recommend to a company wanting to create a brand new app. 

First, I would advise a company to look into the music genre. Even though the popularity of music apps based on their number of installs/reviews is inflated by mainstream music apps, the sheer popularity of the music industry allows for smaller music apps to succeed. People seem to love listening to music, so perhaps we can create an app that will aid people in listening to music. Apps such as song recommenders, tools for making music videos run in the background, or choosing a popular singer/band and making an app about their music would be a couple of ideas. 

Second, I would recommend a health and fitness app. Fitness apps are highly represented in both the App Store and Google Play. People are clearly interested in taking care of their health and fitness, particularly with exercise and calories. 

Lastly, a game app would be another recommendation. We can't ignore the popularity of games. As we said earlier, they are always at the top of the market regardless of the platform. However, because there are so many mobile games out there already, we would have to make sure that our game is unique and enjoyable. Perhaps we can get creative and make a musical game, or even a game that is about health and fitness. The possibilities are endless, but through data analysis we can narrow down those possibilities and make an informed decision based on data. 
