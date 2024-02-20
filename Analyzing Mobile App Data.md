# Analyzing Mobile App Data

In this analysis, we will be looking into a dataset of mobile apps for IOS/Android. We will be looking at apps that are free to download/install on Google Play and the App Store. We will identify which genres and categories are the most popular, and our ultimate goal is to gather insights on what kind of apps will attract the most number of users.  

**Opening and Exploring the Data**

The dataset that will be analyzed is a sample that contains data of about 10,000 Android apps from Google Play, and 7,000 iOS apps from the App Store. The dataset can be accessed through the following links: 

- <a href="https://www.kaggle.com/datasets/lava18/google-play-store-apps">Google Play</a>
- <a href="https://www.kaggle.com/datasets/ramamet4/app-store-apple-data-set-10k-apps">App Store</a>


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

We begin by opening the dataset files and converting them to a list of lists. We then remove the header rows to make it easier for Python to skip over the headers when analyzing the data. We can now start exploring our datasets. 


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
    

A function called `explore_data` was written that will explore the rows and columnms of the data sets in a more readable way. The function also gives us the number of rows and columns in the dataset. We can see that the Google Play data set has 10841 rows and 13 columns. Among the columns, the `App`, `Category`, `Rating`, `Reviews`, `Installs`, `Type`, `Price`, `Content Rating`, and `Genres` columns are the ones that should be noted, as these columns may help us understand and gauge the popularity of certain apps. 


```python
print
print(android[0:3])
```

    [['Photo Editor & Candy Camera & Grid & ScrapBook', 'ART_AND_DESIGN', '4.1', '159', '19M', '10,000+', 'Free', '0', 'Everyone', 'Art & Design', 'January 7, 2018', '1.0.0', '4.0.3 and up'], ['Coloring book moana', 'ART_AND_DESIGN', '3.9', '967', '14M', '500,000+', 'Free', '0', 'Everyone', 'Art & Design;Pretend Play', 'January 15, 2018', '2.0.0', '4.0.3 and up'], ['U Launcher Lite – FREE Live Cool Themes, Hide Apps', 'ART_AND_DESIGN', '4.7', '87510', '8.7M', '5,000,000+', 'Free', '0', 'Everyone', 'Art & Design', 'August 1, 2018', '1.2.4', '4.0.3 and up']]
    

**Deleting Wrong Data**


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
    

**Check for duplicate names:**

The Google Play data set has lots of duplicate entries. We'll be removing rows based on how recent the data entry is. By looking at the reviews for each app, we can know how recent the data is. For example, out of the four duplicate data entries for Instagram, the one with the most reviews will be the most recent data. We will keep that one and remove the other 3.

Instagram is an example of an app that has duplicate entries. It occurs 4 times


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
    

Let's check to see how many duplicate apps we have for android:


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
    


```python
print('Exptected length:', len(android) - 1181)
```

    Exptected length: 9659
    

Loop through the android data set and add each entry to the "reviews_max dictionary." If the entry is already in "reviews_max," the code checks to see if the current entry has a higher number of reviews than the one in the dictionary. The entry with the higher number of reviews is kept. Reviews_max will contain the maximum number of reviews for each app in the dataset. 

**Removing Duplicate Entries: Part Two**


```python
reviews_max = {}

for app in android:
    name = app[0]
    n_reviews = float(app[3])
    
    if name in reviews_max and reviews_max[name] < n_reviews:
        reviews_max[name] = n_reviews
    elif name not in reviews_max:
        reviews_max[name] = n_reviews
        
print(len(reviews_max))
```

    9659
    

The code loops through the dataset and uses the "reviews_max" dictionary as a reference to determine if the data entry that is being examined has the highest number of reviews among its duplicates. If the number of reviews for the data entry matches that of the number of reviews in "reviews_max," this means that it has the highest number of reviews. 

The "name not in already_added" is needed to prevent duplicate entries with the highest number of reviews. At then end, list "android_clean" will contain the entire row of the app's dataset that has the highest number of reviews. 


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


```python
explore_data(android_clean, 0, 4, True)
```

    ['Photo Editor & Candy Camera & Grid & ScrapBook', 'ART_AND_DESIGN', '4.1', '159', '19M', '10,000+', 'Free', '0', 'Everyone', 'Art & Design', 'January 7, 2018', '1.0.0', '4.0.3 and up']
    
    
    ['U Launcher Lite – FREE Live Cool Themes, Hide Apps', 'ART_AND_DESIGN', '4.7', '87510', '8.7M', '5,000,000+', 'Free', '0', 'Everyone', 'Art & Design', 'August 1, 2018', '1.2.4', '4.0.3 and up']
    
    
    ['Sketch - Draw & Paint', 'ART_AND_DESIGN', '4.5', '215644', '25M', '50,000,000+', 'Free', '0', 'Teen', 'Art & Design', 'June 8, 2018', 'Varies with device', '4.2 and up']
    
    
    ['Pixel Draw - Number Art Coloring Book', 'ART_AND_DESIGN', '4.3', '967', '2.8M', '100,000+', 'Free', '0', 'Everyone', 'Art & Design;Creativity', 'June 20, 2018', '1.1', '4.4 and up']
    
    
    Number of rows: 9659
    Number of columns: 13
    

**Removing Non-English Apps**

The dataset has some non-English apps that we want to get rid of. Here are some examples:


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
    

**Isolating the Free Apps**


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
    

**Most Common Apps by Genre**

We want to find an app profile that fits both the App Store and Google Play


**App Store**


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
    

Analysis: 

Based on this data, the most common app category in the App Store is the gaming genre. Entertainment apps come at second place, but the competition is not close. Games are by far the most dominant category in this dataset. 

On a broader scale, apps in the App Store that are designed for entertainment purposes (such as games, social media, photography, etc.) generally tend to be more common than apps designed for practical purposes (such as education, shopping, utilities, productivity, etc.). However, the large number of apps of a certain category does not automatically translate to popularity. The availability of theses apps may not match the demand for them. 

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
    

Analysis: 

In the Google Play Store, the most common app genre is "Family." It is almost double that of the next most common category, which is the "Game" genre. "Tools" genre is a close third place. 

Compared to the App Store data, games are still among the most common apps in Google Play. In fact, the Family genre is composed mostly of games meant for kids. However, they are nowhere near as common as they are in the App Store. It looks like the apps that are geared towards families tend to be more common. 

The dataset here is definitely less skewed towards a single category as it is for the gaming category in the App Store. There is much better representation of practical apps in the Google Play Store. 

**Most Popular Apps by Genre (App Store)**

Here, we are trying to determine the kind of apps with the most users. The App Store data does not provide us with the number of users, so we are going to use the total number of user ratings instead. 


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
    

Based on reviews alone, navigations apps have the highest number of users. However, if we look into the data closely, it is clear that most of these reviews are exclusively for Google Maps and Waze: 


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
    

Google Maps and Waze alone make up for almost 500,000 reviews for navigation apps. It's important to keep in mind that when it comes to certain genres that have a lot of reviews, some of these numbers are skewed by a few apps that happen to be very popular. Here are some examples: 




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
    

**Most Popular Apps by Genre (Google Play)**


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
    

Analysis: 

Based on this data, there is a clear top 3 in the Google Play store: communication, video players, and social media. However, once again these are genres whose data are skewed by a few apps that dominate the market (WhatsApp, Messenger, Skype, etc.). 

A game app seems to be a safe recommendation. No matter the platform, games are always at the top of the market. Another recommendation would be a health and fitness app. Fitness is something that a lot of people seem to be interested in, yet at the same time


```python

```
