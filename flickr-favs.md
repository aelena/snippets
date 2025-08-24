This was an attemp to automatically downlaod copies of all my flickr favorites from other users. They are quite a few.
Unfortunately, there does not seem to be a way around the regrettable design decision that the API will not serve any image labelled unsafe, mostly nsfw stuff.

I originally created this in Google Colab as well, for mere convenience.

```python
# Install flickrapi if not already installed
!pip install flickrapi

# Import necessary libraries
import flickrapi
import requests
import os
import datetime
import re
import shutil
import requests
```

So, for Google Colab also this

```python
from google.colab import userdata
from google.colab import files
```

Set up keys and stuff, if using Colab. Also an entirely optional block for if you want to control the name where images will be downloaded. 

```python
# Create a Flickr API object
api_key = userdata.get('FLICKR_API')
api_secret = userdata.get('FLICKR_SECRET')
flickr = flickrapi.FlickrAPI(api_key, api_secret, format='parsed-json')

if os.path.exists('downloaded_images'):
    print("Directory 'downloaded_images' already exists. Removing...")
    shutil.rmtree('downloaded_images')

if not os.path.exists('downloaded_images'):
    os.makedirs('downloaded_images')
    print("Directory 'downloaded_images' created successfully.")
else:
    print("Directory 'downloaded_images' already exists.")
```

Let's see how much we got

```python
user_id = userdata.get('FLICKR_USERID')
numPages = 1
response = flickr.favorites.getList(user_id=user_id)
print(response)
numPages = response['photos']['pages']
print(f"Number of pages: {numPages}")
```

Produces output like this

```
{'photos': {'page': 1, 'pages': 104, 'perpage': 100, 'total': 10337, 'photo': [{'id': '54085533846', 'owner': '18583713@N06', 'secret': '1835539905', 'server': '65535', 'farm': 66, 'title': 'false horizons', 'ispublic': 1, 'isfriend': 0, 'isfamily': 0, 'date_faved': '1729606824'}, {'id': '54060925898', 'owner': '150720244@N06', 'secret': '564f29d7d1', 'server': '65535', 'farm': 66, 'title': 'De Banken', 'ispublic': 1, 'isfriend': 0, 'isfamily': 0, 'date_faved': '1728717774'}, {'id': '54051788389', 'owner': '150720244@N06', 'secret': '8ae77a3d89', 'server': '65535', 'farm': 66, 'title': 'Feather.', 'ispublic': 1, 'isfriend': 0, 'isfamily': 0, 'date_faved': '1728456758'}, {'id': '53943639076', 'owner': '150720244@N06', 'secret': 'a3f015185c', 'server': '65535', 'farm': 66, 'title': 'Haze.', 'ispublic': 1, 'isfriend': 0, 'isfamily': 0, 'date_faved': '1727968839'}, {'id': '54030692810', 'owner': '59509464@N00', 'secret': '7033e0fc0c', 'server': '65535', 'farm': 66, 'title': 'Klettar', 'ispublic': 1, 'isfriend': 0, 'isfamily': 0, 'date_faved': '1727635326'}, {'id': '54024647857', 'owner': '182557475@N08', 'secret': '0e901d90d1', 'server': '65535', 'farm': 66, 'title': '', 'ispublic': 1, 'isfriend': 0, 'isfamily': 0, 'date_faved': '1727633392'}, {'id': '54026046052', 'owner': '47062778@N06', 'secret': '9cb7f22018', 'server': '65535', 'farm': 66, 'title': 'Morning Splash', 'ispublic': 1, 'isfriend': 0, 'isfamily': 0, 'date_faved': '1727633376'}, {'id': '53947039261', 'owner': '19685823@N00', 'secret': 'c899dfbfcd', 'server': '65535', 'farm': 66, 'title': 'Überreste der Cassonsbahn-Bergstation | Fil da Cassons . Flims (GR)', 'ispublic': 1, 'isfriend': 0, 'isfamily': 0, 'date_faved': '1727633362'}, {'id': '54029912323', 'owner': '18583713@N06', 'secret': '218ee3f29d', 'server': '65535', 'farm': 66, 'title': '~Sea of Japan~', 'ispublic': 1, 'isfriend': 0, 'isfamily': 0, 'date_faved': '1727633298'}, {'id': '48629694776', 'owner': '41765319@N00', 'secret': '74669f4f41', 'server': '65535', 'farm': 66, 'title': 'Atmospheric Interlude', 'ispublic': 1, 'isfriend': 0, 'isfamily': 0, 'date_faved': '1725617319'}, {'id': '22886215719', 'owner': '98637685@N03', 'secret': '1d949800de', 'server': '563', 'farm': 1, 'title': 'Atmospheric', 'ispublic': 1, 'isfriend': 0, 'isfamily': 0, 'date_faved': '1725617303'}, {'id': '31724509538', 'owner': '115562909@N03', 'secret': '08ba46df11', 'server': '1937', 'farm': 2, 'title': 'Moody', 'ispublic': 1, 'isfriend': 0, 'isfamily': 0, 'date_faved': '1725616111'}, {'id': '53972047727', 'owner': '63079447@N07', 'secret': 'dd56021c3c', 'server': '65535', 'farm': 66, 'title': 'mani-3588', 'ispublic': 1, 'isfriend': 0, 'isfamily': 0, 'date_faved': '1725616064'}, {'id': '8100091129', 'owner': '61237922@N05', 'secret': '666d1507e6', 'server': '8328', 'farm': 9, 'title': '启明  /  Phospherus', 'ispublic': 1, 'isfriend': 0, 'isfamily': 0, 'date_faved': '1724495794'}, {'id': '9004595851', 'owner': '98866476@N00', 'secret': 'a79855488a', 'server': '3797', 'farm': 4, 'title': '', 'ispublic': 1, 'isfriend': 0, 'isfamily': 0, 'date_faved': '1724495777'}, {'id': '17757566743', 'owner': '9814826@N05', 'secret': 'e47909ee00', 'server': '296', 'farm': 1, 'title': 'Athens Pano 2', 'ispublic': 1, 'isfriend': 0, 'isfamily': 0, 'date_faved': '1724314789'}, {'id': '14634484507', 'owner': '41302121@N05', 'secret': '5cdda87d6e', 'server': '5592', 'farm': 6, 'title': 'Athens', 'ispublic': 1, 'isfriend': 0, 'isfamily': 0, 'date_faved': '1724314762'}, {'id': '40152238463', 'owner': '31611900@N00', 'secret': 'd863780ddc', 'server': '7888', 'farm': 8, 'title': 'athens', 'ispublic': 1, 'isfriend': 0, 'isfamily': 0, 'date_faved': '1724314452'}, {'id': '51097069562', 'owner': '54544405@N04', 'secret': '8b567aa92d', 'server': '65535', 'farm': 66, 'title': 'MATRIX', 'ispublic': 1, 'isfriend': 0, 'isfamily': 0, 'date_faved': '1724314112'}, {'id': '52259464442', 'owner': '54544405@N04', 'secret': '25b7fe72ee', 'server': '65535', 'farm': 66, 'title': "JUNGLES DON'T ALWAYS HAVE TREES", 'ispublic': 1, 'isfriend': 0, 'isfamily': 0, 'date_faved': '1724314081'}, {'id': '16282616371', 'owner': '21089324@N02', 'secret': '5acc53e549', 'server': '8621', 'farm': 9, 'title': 'Tromso', 'ispublic': 1, 'isfriend': 0, 'isfamily': 0, 'date_faved': '1724314045'}, {'id': '16562824133', 'owner': '55248767@N05', 'secret': '741f082cb0', 'server': '7694', 'farm': 8, 'title': 'Paris', 'ispublic': 1, 'isfriend': 0, 'isfamily': 0, 'date_faved': '1724313943'}, {'id': '20827650980', 'owner': '88929764@N00', 'secret': 'abd08970e2', 'server': '643', 'farm': 1, 'title': 'Out Alee', 'ispublic': 1, 'isfriend': 0, 'isfamily': 0, 'date_faved': '1724313880'}, {'id': '46338309165', 'owner': '51400837@N07', 'secret': '93efa48483', 'server': '7804', 'farm': 8, 'title': '', 'ispublic': 1, 'isfriend': 0, 'isfamily': 0, 'date_faved': '1724313825'}, {'id': '47425728272', 'owner': '51400837@N07', 'secret': '168be557a7', 'server': '7900', 'farm': 8, 'title': '', 'ispublic': 1, 'isfriend': 0, 'isfamily': 0, 'date_faved': '1724313789'}, {'id': '788454067', 'owner': '45767439@N00', 'secret': '646399c370', 'server': '1003', 'farm': 2, 'title': 'Pinhole#17', 'ispublic': 1, 'isfriend': 0, 'isfamily': 0, 'date_faved': '1724311478'}, {'id': '895860881', 'owner': '45767439@N00', 'secret': '6018beacb5', 'server': '1395', 'farm': 2, 'title': 'Pinhole#25', 'ispublic': 1, 'isfriend': 0, 'isfamily': 0, 'date_faved': '1724311474'}, {'id': '8560562428', 'owner': '20914166@N00', 'secret': 'c5d2d3b213', 'server': '8373', 'farm': 9, 'title': 'Blue', 'ispublic': 1, 'isfriend': 0, 'isfamily': 0, 'date_faved': '1724311436'}, {'id': '8644128676', 'owner': '30419622@N05', 'secret': 'efcecffff9', 'server': '8110', 'farm': 9, 'title': '', 'ispublic': 1, 'isfriend': 0, 'isfamily': 0, 'date_faved': '1724311388'}, {'id': '14520200786', 'owner': '57139324@N00', 'secret': 'fd51defd56', 'server': '3912', 'farm': 4, 'title': 'l a m a n c h e', 'ispublic': 1, 'isfriend': 0, 'isfamily': 0, 'date_faved': '1724311357'}, {'id': '13919489897', 'owner': '57139324@N00', 'secret': 'edf3dcdf7b', 'server': '7427', 'farm': 8, 'title': 'b r i g h t o n', 'ispublic': 1, 'isfriend': 0, 'isfamily': 0, 'date_faved': '1724311343'}, {'id': '15528837950', 'owner': '57139324@N00', 'secret': '9b8bdc2be1', 'server': '7526', 'farm': 8, 'title': 's u n', 'ispublic': 1, 'isfriend': 0, 'isfamily': 0, 'date_faved': '1724311331'}, {'id': '48062786993', 'owner': '57139324@N00', 'secret': '454f74d2f9', 'server': '65535', 'farm': 66, 'title': 'e v e r y t h i n g i s g r e e n', 'ispublic': 1, 'isfriend': 0, 'isfamily': 0, 'date_faved': '1724311287'}, {'id': '53937827251', 'owner': '18583713@N06', 'secret': '93bceebbbd', 'server': '65535', 'farm': 66, 'title': 'summer showers', 'ispublic': 1, 'isfriend': 0, 'isfamily': 0, 'date_faved': '1724310814'}, {'id': '53186407006', 'owner': '44470682@N04', 'secret': '748569dfe3', 'server': '65535', 'farm': 66, 'title': '4108', 'ispublic': 1, 'isfriend': 0, 'isfamily': 0, 'date_faved': '1723498102'}, {'id': '53909535598', 'owner': '150720244@N06', 'secret': '0b56a1f18e', 'server': '65535', 'farm': 66, 'title': 'De Banken', 'ispublic': 1, 'isfriend': 0, 'isfamily': 0, 'date_faved': '1723389621'}, {'id': '53911918160', 'owner': '150720244@N06', 'secret': '65b6992827', 'server': '65535', 'farm': 66, 'title': 'Waves.', 'ispublic': 1, 'isfriend': 0, 'isfamily': 0, 'date_faved': '1723389560'}, {'id': '53908972477', 'owner': '182557475@N08', 'secret': '0314308e7d', 'server': '65535', 'farm': 66, 'title': '', 'ispublic': 1, 'isfriend': 0, 'isfamily': 0, 'date_faved': '1723154190'}, {'id': '52526305674', 'owner': '111631195@N05', 'secret': 'ec1c48bcef', 'server': '65535', 'farm': 66, 'title': 'NOTTE A ESTE', 'ispublic': 1, 'isfriend': 0, 'isfamily': 0, 'date_faved': '1723154138'}, {'id': '25649006054', 'owner': '111631195@N05', 'secret': '454ea3ef35', 'server': '1688', 'farm': 2, 'title': 'WOMAN ON BICYCLE WITH GREEN COAT AND UMBRELLA', 'ispublic': 1, 'isfriend': 0, 'isfamily': 0, 'date_faved': '1723154125'}, {'id': '53760234097', 'owner': '134378250@N05', 'secret': 'dc204fc194', 'server': '65535', 'farm': 66, 'title': 'Hamburg Classic II', 'ispublic': 1, 'isfriend': 0, 'isfamily': 0, 'date_faved': '1722857674'}, {'id': '53763419969', 'owner': '137489628@N06', 'secret': '4c54b7c24c', 'server': '65535', 'farm': 66, 'title': 'Happy birthday to first 1000 years! explored Nr. 5 😀', 'ispublic': 1, 'isfriend': 0, 'isfamily': 0, 'date_faved': '1722857657'}, {'id': '53898029882', 'owner': '169854405@N05', 'secret': 'f2ed3e8240', 'server': '65535', 'farm': 66, 'title': 'Photo and Copyright by Manolis Anastasiadis ©', 'ispublic': 1, 'isfriend': 0, 'isfamily': 0, 'date_faved': '1722857345'}, {'id': '53849728155', 'owner': '129894947@N04', 'secret': '23c7450158', 'server': '65535', 'farm': 66, 'title': 'Birds Eye | Hong Kong', 'ispublic': 1, 'isfriend': 0, 'isfamily': 0, 'date_faved': '1722857240'}, {'id': '53897940407', 'owner': '150667648@N06', 'secret': '946db3d3a1', 'server': '65535', 'farm': 66, 'title': 'DSC_0529', 'ispublic': 1, 'isfriend': 0, 'isfamily': 0, 'date_faved': '1722708440'}, {'id': '53034326836', 'owner': '12248978@N07', 'secret': '4722fb55ca', 'server': '65535', 'farm': 66, 'title': 'Kjerag norway 090723', 'ispublic': 1, 'isfriend': 0, 'isfamily': 0, 'date_faved': '1722505485'}, {'id': '1801831168', 'owner': '17974769@N00', 'secret': 'c8610cadca', 'server': '2047', 'farm': 3, 'title': 'Omnia vincit amor, et nos cedamus amori', 'ispublic': 1, 'isfriend': 0, 'isfamily': 0, 'date_faved': '1722505321'}, {'id': '2995218058', 'owner': '77053677@N00', 'secret': '10828e314b', 'server': '3239', 'farm': 4, 'title': '~ The Salmon River Chronicles (Part II)', 'ispublic': 1, 'isfriend': 0, 'isfamily': 0, 'date_faved': '1722505314'}, {'id': '23541219139', 'owner': '27910428@N08', 'secret': 'a259d34bba', 'server': '691', 'farm': 1, 'title': 'Soothing', 'ispublic': 1, 'isfriend': 0, 'isfamily': 0, 'date_faved': '1722505284'}, {'id': '52897723608', 'owner': '27910428@N08', 'secret': '067e3d1ed1', 'server': '65535', 'farm': 66, 'title': 'Reality Turning Into A Dream', 'ispublic': 1, 'isfriend': 0, 'isfamily': 0, 'date_faved': '1722505260'}, {'id': '53889629413', 'owner': '49587636@N05', 'secret': 'a38543047a', 'server': '65535', 'farm': 66, 'title': 'Gold', 'ispublic': 1, 'isfriend': 0, 'isfamily': 0, 'date_faved': '1722415681'}, {'id': '53883043460', 'owner': '43684069@N05', 'secret': '36d5be9531', 'server': '65535', 'farm': 66, 'title': 'Spitsbergen', 'ispublic': 1, 'isfriend': 0, 'isfamily': 0, 'date_faved': '1722413116'}, {'id': '53883850286', 'owner': '86145600@N07', 'secret': '90c9c42f2f', 'server': '65535', 'farm': 66, 'title': 'Classic Wabi-Sabi aesthetics', 'ispublic': 1, 'isfriend': 0, 'isfamily': 0, 'date_faved': '1722350097'}, {'id': '53420034454', 'owner': '104359226@N07', 'secret': '39669ffc0e', 'server': '65535', 'farm': 66, 'title': 'BW #2', 'ispublic': 1, 'isfriend': 0, 'isfamily': 0, 'date_faved': '1722349902'}, {'id': '53588993209', 'owner': '150720244@N06', 'secret': '2e8a2f3732', 'server': '65535', 'farm': 66, 'title': 'Glow', 'ispublic': 1, 'isfriend': 0, 'isfamily': 0, 'date_faved': '1722349848'}, {'id': '53748962031', 'owner': '150720244@N06', 'secret': '607253707b', 'server': '65535', 'farm': 66, 'title': 'Dance', 'ispublic': 1, 'isfriend': 0, 'isfamily': 0, 'date_faved': '1722349833'}, {'id': '53766068850', 'owner': '150720244@N06', 'secret': 'bca8e72bbe', 'server': '65535', 'farm': 66, 'title': 'Low', 'ispublic': 1, 'isfriend': 0, 'isfamily': 0, 'date_faved': '1722349827'}, {'id': '53795572192', 'owner': '150720244@N06', 'secret': 'cefc20e180', 'server': '65535', 'farm': 66, 'title': 'Coast.', 'ispublic': 1, 'isfriend': 0, 'isfamily': 0, 'date_faved': '1722349819'}, {'id': '53824883071', 'owner': '150720244@N06', 'secret': '6825fc3b97', 'server': '65535', 'farm': 66, 'title': 'Parts.', 'ispublic': 1, 'isfriend': 0, 'isfamily': 0, 'date_faved': '1722349816'}, {'id': '53873089265', 'owner': '150720244@N06', 'secret': '2ae2150ffc', 'server': '65535', 'farm': 66, 'title': 'Coast', 'ispublic': 1, 'isfriend': 0, 'isfamily': 0, 'date_faved': '1722349805'}, {'id': '53877869204', 'owner': '150720244@N06', 'secret': '76048ecb9f', 'server': '65535', 'farm': 66, 'title': 'Scape', 'ispublic': 1, 'isfriend': 0, 'isfamily': 0, 'date_faved': '1722349801'}, {'id': '53891088155', 'owner': '135291551@N06', 'secret': 'd743a76c7a', 'server': '65535', 'farm': 66, 'title': 'Unloading', 'ispublic': 1, 'isfriend': 0, 'isfamily': 0, 'date_faved': '1722341801'}, {'id': '45089089924', 'owner': '21058160@N07', 'secret': '98e1619988', 'server': '4875', 'farm': 5, 'title': 'Torridon Hills', 'ispublic': 1, 'isfriend': 0, 'isfamily': 0, 'date_faved': '1722334316'}, {'id': '51690286233', 'owner': '193099293@N07', 'secret': 'e6fb9cdd7b', 'server': '65535', 'farm': 66, 'title': 'School House', 'ispublic': 1, 'isfriend': 0, 'isfamily': 0, 'date_faved': '1722334059'}, {'id': '48744476931', 'owner': '128807045@N08', 'secret': '815b8cd46e', 'server': '65535', 'farm': 66, 'title': 'G A R R O N • B R I D G E', 'ispublic': 1, 'isfriend': 0, 'isfamily': 0, 'date_faved': '1722334030'}, {'id': '46842168961', 'owner': '125791209@N06', 'secret': '038f55a414', 'server': '7879', 'farm': 8, 'title': 'Another Scottish Black and White', 'ispublic': 1, 'isfriend': 0, 'isfamily': 0, 'date_faved': '1722333952'}, {'id': '53852065285', 'owner': '126617331@N06', 'secret': 'd843fd9c75', 'server': '65535', 'farm': 66, 'title': 'Série 83: Chemins (2)', 'ispublic': 1, 'isfriend': 0, 'isfamily': 0, 'date_faved': '1722333910'}, {'id': '53658882502', 'owner': '158767070@N04', 'secret': 'd43e305a37', 'server': '65535', 'farm': 66, 'title': 'Hooksiel, Strandhaus 1, 14.4.2024', 'ispublic': 1, 'isfriend': 0, 'isfamily': 0, 'date_faved': '1722333481'}, {'id': '53882639374', 'owner': '158767070@N04', 'secret': 'ae108c6183', 'server': '65535', 'farm': 66, 'title': 'Muschelsucher - Shell Seekers', 'ispublic': 1, 'isfriend': 0, 'isfamily': 0, 'date_faved': '1722333403'}, {'id': '53855576366', 'owner': '47564323@N07', 'secret': '7865a1c4ef', 'server': '65535', 'farm': 66, 'title': 'Border', 'ispublic': 1, 'isfriend': 0, 'isfamily': 0, 'date_faved': '1722319026'}, {'id': '53231722730', 'owner': '198821882@N03', 'secret': '3d4beb3ca8', 'server': '65535', 'farm': 66, 'title': 'An caisteal air an oirthir', 'ispublic': 1, 'isfriend': 0, 'isfamily': 0, 'date_faved': '1722318533'}, {'id': '53692234939', 'owner': '13684881@N06', 'secret': 'abd8368b99', 'server': '65535', 'farm': 66, 'title': 'it scares me how temporary everything is', 'ispublic': 1, 'isfriend': 0, 'isfamily': 0, 'date_faved': '1722318463'}, {'id': '53704900764', 'owner': '33996498@N00', 'secret': 'da290dac15', 'server': '65535', 'farm': 66, 'title': 'Le nuage', 'ispublic': 1, 'isfriend': 0, 'isfamily': 0, 'date_faved': '1722318431'}, {'id': '53885711822', 'owner': '198821882@N03', 'secret': 'bd4e44d3cf', 'server': '65535', 'farm': 66, 'title': 'Sleeper in Metropolis', 'ispublic': 1, 'isfriend': 0, 'isfamily': 0, 'date_faved': '1722282409'}, {'id': '53800410657', 'owner': '151833726@N07', 'secret': 'c3680441ea', 'server': '65535', 'farm': 66, 'title': 'Inhumain', 'ispublic': 1, 'isfriend': 0, 'isfamily': 0, 'date_faved': '1722277589'}, {'id': '53888845985', 'owner': '137551550@N05', 'secret': 'd80e7fe7ce', 'server': '65535', 'farm': 66, 'title': 'Iceland', 'ispublic': 1, 'isfriend': 0, 'isfamily': 0, 'date_faved': '1722277557'}, {'id': '53888761219', 'owner': '137551550@N05', 'secret': '489d77cd85', 'server': '65535', 'farm': 66, 'title': 'Iceland', 'ispublic': 1, 'isfriend': 0, 'isfamily': 0, 'date_faved': '1722277552'}, {'id': '53887514272', 'owner': '137551550@N05', 'secret': '7209ce25de', 'server': '65535', 'farm': 66, 'title': 'Iceland', 'ispublic': 1, 'isfriend': 0, 'isfamily': 0, 'date_faved': '1722277544'}, {'id': '53269862536', 'owner': '145139001@N04', 'secret': '75f1c9458e', 'server': '65535', 'farm': 66, 'title': 'Bild 3 Der Leuchtturm', 'ispublic': 1, 'isfriend': 0, 'isfamily': 0, 'date_faved': '1722191318'}, {'id': '53275113722', 'owner': '145139001@N04', 'secret': 'd833525bd5', 'server': '65535', 'farm': 66, 'title': 'Leuchtturmpanorama', 'ispublic': 1, 'isfriend': 0, 'isfamily': 0, 'date_faved': '1722191311'}, {'id': '53666610597', 'owner': '194715158@N06', 'secret': '515d67e15e', 'server': '65535', 'farm': 66, 'title': '', 'ispublic': 1, 'isfriend': 0, 'isfamily': 0, 'date_faved': '1722173237'}, {'id': '52665203658', 'owner': '78803005@N05', 'secret': 'fcff19f37e', 'server': '65535', 'farm': 66, 'title': '', 'ispublic': 1, 'isfriend': 0, 'isfamily': 0, 'date_faved': '1722173133'}, {'id': '52641215040', 'owner': '78803005@N05', 'secret': '486fd5a07b', 'server': '65535', 'farm': 66, 'title': 'Through the curtains', 'ispublic': 1, 'isfriend': 0, 'isfamily': 0, 'date_faved': '1722173128'}, {'id': '52789302702', 'owner': '134179010@N03', 'secret': 'c5aee6c974', 'server': '65535', 'farm': 66, 'title': 'Cadini di Misurina Sunset', 'ispublic': 1, 'isfriend': 0, 'isfamily': 0, 'date_faved': '1722173005'}, {'id': '53719665236', 'owner': '53887853@N03', 'secret': '8c4c7b374c', 'server': '65535', 'farm': 66, 'title': 'Unspoiled beach', 'ispublic': 1, 'isfriend': 0, 'isfamily': 0, 'date_faved': '1722172826'}]}, 'stat': 'ok'}
Number of pages: 104
```

Get the date format and name for the downloaded photo, as per my strictly personal preference. YMMV.
```python
def get_dt_from_photo(photo):
  # Extract the 'date_faved' timestamp from the photo dictionary
  dt = datetime.datetime.fromtimestamp(int(photo['date_faved']))
  dt = dt.strftime('%Y%m%d_%H%M%S')
  return dt

def get_saved_filename(photo):
  dt = get_dt_from_photo(photo)

  # Get the photo title and remove any non-alphanumeric characters (except spaces)
  title = re.sub(r'[^\w\s]', '', photo['title'])
  if len(title) > 32:
    title = title[:32]

  # Combine the formatted date and cleaned title to create a filename, just my preference here
  return f"{dt}___{title}"
```

Get best size available, depending on what owners have uploaded.

```python
def get_best_size(sizes):
    """
    Takes a list of photo sizes from the Flickr API and returns
    the preferred size and its source URL based on a predefined preference list.
    always prioritizong larger sizes like 'Original', 'Large 2048', etc.
    The mappings are what flickr uses to denote sizes.
    """
    try:

      # sizes returned by photo info come from smaller (thumbnail)
      # to original, so we reverse in place as we are interested
      # in the first best size
      sizes.reverse()
      # Define the size preference list in order of preference
      size_preference = ['o', 'k', 'h', 'b', 'c', 'z', 'm', 'n', 'w', 't', 'q', 's']
      # Mapping of size labels to their descriptive labels
      size_label_mapping = {
          'o': 'Original',
          'k': 'Large 2048',
          'h': 'Large 1600',
          'b': 'Large 1024',
          'c': 'Medium 800',
          'z': 'Medium 640',
          'n': 'Small 320',
          'm': 'Small 240',
          't': 'Thumbnail 100',
          'q': 'Large Square 150',
          's': 'Small Square 75'
      }

      for preferred_size in size_preference:
          # Get the corresponding label from the mapping
          label_to_find = size_label_mapping[preferred_size]
          for size in sizes:
              # Check if the label matches the mapped size label
              if label_to_find.lower() in size['label'].lower():
                  return (preferred_size, size['source'])

      return None  # No preferred sizes found

    except:
        print(f"ERROR : Error occurred")
```

A little function to download from the API

```python
def download_image(image_url, save_path):
    try:
        # Send a HTTP GET request to the image URL
        response = requests.get(image_url)
        # Check if the request was successful (status code 200)
        if response.status_code == 200:
            # Open a file in write-binary mode to save the image
            with open(save_path, 'wb') as file:
                file.write(response.content)
            print(f"INFO : Image successfully downloaded: {save_path}")
        else:
            print(f"ERROR : Failed to download {save_path} from {image_url}. Status code: {response.status_code}")

    except requests.RequestException as e:
        print(f"ERROR : Error occurred: {e}")
```

This is sort of the minimal code necessary to hit the API to download stuff, with some printing out progress for readability and tracking

```python
for page in range(0, 2):

  print("------------------------------------------------------------------------")
  print(f"INFO : DOWNLOADING PAGE : {page}")

  photos = flickr.favorites.getList(user_id=user_id, per_page=100, page=page, safe_search='3')

  for i, photo in enumerate(photos['photos']['photo']):
    # print(f"Title: {photo['title']}, Id: {photo['id']}, Date faved: {photo['date_faved']}")
    print(photo)
```

Produces ouput that looks like this, with a line for each photo that is successfully downloaded

```
------------------------------------------------------------------------
INFO : DOWNLOADING PAGE : 0
{'id': '54042439615', 'owner': '91756830@N00', 'secret': 'f2d5cff8a9', 'server': '65535', 'farm': 66, 'title': 'Vaunt', 'ispublic': 1, 'isfriend': 0, 'isfamily': 0, 'date_faved': '1729753667'}
{'id': '54085533846', 'owner': '18583713@N06', 'secret': '1835539905', 'server': '65535', 'farm': 66, 'title': 'false horizons', 'ispublic': 1, 'isfriend': 0, 'isfamily': 0, 'date_faved': '1729606824'}
{'id': '54060925898', 'owner': '150720244@N06', 'secret': '564f29d7d1', 'server': '65535', 'farm': 66, 'title': 'De Banken', 'ispublic': 1, 'isfriend': 0, 'isfamily': 0, 'date_faved': '1728717774'}
...
```

This makes use of getting the best size available and really downloading the image and saving it to our specified folder, that we indicated earler.

```python
photo_size = 'b'

# for page in range(0, numPages+1):
for page in range(0, 1):

  print("------------------------------------------------------------------------")
  print(f"INFO : DOWNLOADING PAGE : {page}")

  photos = flickr.favorites.getList(user_id=user_id, per_page=100, page=page)

  for i, photo in enumerate(photos['photos']['photo']):

    # if photo['id'] != '53394024836':
    #   continue

    print("------------------------------------------------------------------------")

    if(photo['ispublic'] != 1):
      print(f"{photo['title']} is not public")
      continue;

    # check license, a value of 0 means all rights reserved
    # license = flickr.photos.getInfo(photo_id=photo['id'])
    # license = license['photo']['license']

    # check what sizes we have for the photo and if the Original size
    # is available (photos that cannot be downloaded do not have the original,
    # you can have original and still license = 0, so the settings are not
    # related)
    sizes = flickr.photos.getSizes(photo_id=photo['id'])
    sizes = sizes['sizes']['size']
    photo_size = get_best_size(sizes)
    if photo_size == None:
      print(f"WARN : Skipping {photo['title']} due to no good size")
      continue
    else:
      print(f"DEBUG : {photo['title']} with size '{photo_size[0]}' and source {photo_size[1]}")
      url = photo_size[1]
      save_filename = get_saved_filename(photo)
      download_image(url, f"downloaded_images/{save_filename}.jpg")

```

Produces output like this

```
  adding: downloaded_images/20070731_110519___kasukabefog6.jpg (deflated 23%)
  adding: downloaded_images/20120210_140928___Osaka night scape.jpg (deflated 1%)
  adding: downloaded_images/20100507_085137___densecity 23.jpg (deflated 0%)
...
```

____

# Bonus
I fancied myself clever, and thought, I will also have them uploaded to OneDrive. Unfortunately, this will work only if you have a SPO license. I was getting this lovely `enant does not have a SPO license` because basically, my tenant is poor.
However, here is the code in case you do have an SPO. I could have changed to Google Drive or whatever, but I never got round to it. Maybe some other day.

### Set things up 
for interaction with 1DRV

```python
!pip install msal requests

import msal

# Azure AD app credentials
CLIENT_ID = userdata.get('1DRV_CLID')
TENANT_ID = userdata.get('AZ_TENANT')
CLIENT_SECRET = userdata.get('1DRV_SECRET')
AUTHORITY = f"https://login.microsoftonline.com/{TENANT_ID}"
USER_ID = userdata.get('USER_ID')

# Scopes for OneDrive access
SCOPES = ["https://graph.microsoft.com/.default"]
# OneDrive upload URL
ONEDRIVE_UPLOAD_URL = f"https://graph.microsoft.com/v1.0/{USER_ID}/drive/root:/"
```

You probably will need to change secret management if running this somewhere else other than Google Colab.

The code that should have worked...

```python


app = msal.ConfidentialClientApplication(
    authority=AUTHORITY,
    client_id=CLIENT_ID,  # Use the CLIENT_ID from your environment
    client_credential=CLIENT_SECRET  # Use the CLIENT_SECRET from your environment
)

def get_access_token():
    result = None
    # First check if there are cached tokens
    accounts = app.get_accounts()
    if accounts:
        result = app.acquire_token_silent(SCOPES, account=accounts[0])

    if not result:
        # If no cached token, request a new one
        result = app.acquire_token_for_client(scopes=SCOPES)

    print(result)

    if "access_token" in result:
        return result["access_token"]
    else:
        raise Exception("Failed to get access token")

def upload_file_to_onedrive(file_path, folder_path, destination_name):
    # Get access token
    access_token = get_access_token()

    # Open the file
    with open(file_path, 'rb') as file_data:
        # Construct the upload URL and headers
        upload_url = f"{ONEDRIVE_UPLOAD_URL}{folder_path}{destination_name}:/content"
        headers = {
            "Authorization": f"Bearer {access_token}",
            "Content-Type": "application/octet-stream"
        }

        # Send the PUT request to upload the file
        response = requests.put(upload_url, headers=headers, data=file_data)

        if response.status_code == 201:
            print(f"File '{destination_name}' uploaded successfully to OneDrive.")
        else:
            print(f"Failed to upload file. Status Code: {response.status_code}")
            print(response.json())


for download in os.listdir('./downloaded_images'):
  if download.startswith('.'):
    continue;
  upload_file_to_onedrive(f"./downloaded_images/{download}", 'Pictures/flickr-faves/', download)

```









