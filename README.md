# Project 3: Google Analytics ETL: Overview 
## Still in progress, comments are enabled if anyone knows how to help 

* The purpose of this project is to become familiar with using API's and building pipelines. The goal is to send viewer data from this repository to Google Cloud. In Google Cloud I set up a service account, and I enabled Admin privledges in my Google Analytics account. Once the data is extracted into Python, I will transform it and send it back to Google Cloud where I can build a dashboard. 

## Progress: 

* I've managed to set up my repository to send data over to Google Analytics, and I've built a Python script that processes the data. Unfortunately, whenever I try to view the data I come up with the same error code. 

'''
HttpError: <HttpError 403 when requesting https://analyticsreporting.googleapis.com/v4/reports:batchGet?alt=json returned "User does not have sufficient permissions for this profile.". Details: "User does not have sufficient permissions for this profile."> 
'''

* I've quadruple checked the service account was created properly, added properly, and the privledges are correct. I will leave my code thus far in the meantime, with the following redacted for privacy. 

'''
your_view_id = '####'
ga_keys = 'user_key.json'
''' 

* Thank you in advance for your help!

