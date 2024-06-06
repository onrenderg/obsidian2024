## NamingStDatafiles & MongoDataProvidr

  
# OrgWay=

##############

# Root

## SubRoot

###############

  

############## MongoDataProvider ############### || NotesRootHeading

#0

## DevRules

### Structure Map by practice and noteMaking of implemented stuff with full project notes for reference and Structre Map Optimization

  

#1.Setup/Root

##MongoServerSetup

## RoboMongoSetup

  

#2.CsvMongoLoaderFunction/Root ||Module

##v1.rCsvFilesNames=DirName+*preDirStructure* ||Component

##v1.rLoadedCollection=CsvFilesNames || Component

```python

!pip install pymongo

import pymongo

  

myClient = pymongo.MongoClient('mongodb://localhost:27017/')

  

myDb = myClient['demoDatabase']

  

#Collection name demoDatabase

tickTable = myDb['demoDatabase']

  

# Passing data to tickTable Object in a callback function

  

# Feed return json with format of dict of dicts as :

'''

{

{'open':'1', 'high':'1','low':'1', 'close':'1','volume':'1', 'instn':''},

{'open':'1', 'high':'1','low':'1', 'close':'','volume':'1', 'instn':''},

...

  

}

'''

# Json insertion in mongoDb #Prompt

def on_message(msg):

msg['instrument'] = msg['insrument'].symbol

tickTable.insert_one(msg#json#)

print(msg)

  

'''

#?

datetime.date(2024,03,01)

'''

  

```

  

##v1.rCsvFromDb

  

###v1.Prj.Binance/GateWsFeedPrj "can remove columns before loading from db rather using pandas for that "

  

### TchGargon

  

Quote/Tick