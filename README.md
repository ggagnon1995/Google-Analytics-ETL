# Project 3: Google Analytics ETL: Overview 
## Still in progress, comments are enabled if anyone knows how to help 

* The purpose of this project is to become familiar with using API's and building pipelines. The goal is to send viewer data from this repository to Google Cloud. In Google Cloud I set up a service account, and I enabled Admin privledges in my Google Analytics account. Once the data is extracted into Python, I will transform it and send it back to Google Cloud where I can build a dashboard. 

## Progress: 

* I've managed to set up my repository to send data over to Google Analytics, and I've built a Python script that processes the data. Unfortunately, whenever I try to view the data I come up with the same error code. 

```python
HttpError: <HttpError 403 when requesting https://analyticsreporting.googleapis.com/v4/reports:batchGet?alt=json returned "User does not have sufficient permissions for this profile.". Details: "User does not have sufficient permissions for this profile."> 
```

* I've quadruple checked the service account was created properly, added properly, and the privledges are correct. I will leave my code thus far in the meantime, with the following redacted for privacy. 

```python
your_view_id = '####'
ga_keys = 'user_key.json'
```

* Thank you in advance for your help!

```python 
import numpy as np 
import pandas as pd
from google.oauth2 import service_account 
## run "pip install google" then run "pip install --upgrade google-api-python-client" in the terminal window
from apiclient.discovery import build
```

```python 
def format_summary(response):
    try:
        # create row index
        try: 
            row_index_names = response['reports'][0]['columnHeader']['dimensions']
            row_index = [ element['dimensions'] for element in response['reports'][0]['data']['rows'] ]
            row_index_named = pd.MultiIndex.from_arrays(np.transpose(np.array(row_index)), 
                                                        names = np.array(row_index_names))
        except:
            row_index_named = None
        
        # extract column names
        summary_column_names = [item['name'] for item in response['reports'][0]
                                ['columnHeader']['metricHeader']['metricHeaderEntries']]
    
        # extract table values
        summary_values = [element['metrics'][0]['values'] for element in response['reports'][0]['data']['rows']]
    
        # combine. I used type 'float' because default is object, and as far as I know, all values are numeric
        df = pd.DataFrame(data = np.array(summary_values), 
                          index = row_index_named, 
                          columns = summary_column_names).astype('float')
    
    except:
        df = pd.DataFrame()
        
    return df

def format_pivot(response):
    try:
        # extract table values
        pivot_values = [item['metrics'][0]['pivotValueRegions'][0]['values'] for item in response['reports'][0]
                        ['data']['rows']]
        
        # create column index
        top_header = [item['dimensionValues'] for item in response['reports'][0]
                      ['columnHeader']['metricHeader']['pivotHeaders'][0]['pivotHeaderEntries']]
        column_metrics = [item['metric']['name'] for item in response['reports'][0]
                          ['columnHeader']['metricHeader']['pivotHeaders'][0]['pivotHeaderEntries']]
        array = np.concatenate((np.array(top_header),
                                np.array(column_metrics).reshape((len(column_metrics),1))), 
                               axis = 1)
        column_index = pd.MultiIndex.from_arrays(np.transpose(array))
        
        # create row index
        try:
            row_index_names = response['reports'][0]['columnHeader']['dimensions']
            row_index = [ element['dimensions'] for element in response['reports'][0]['data']['rows'] ]
            row_index_named = pd.MultiIndex.from_arrays(np.transpose(np.array(row_index)), 
                                                        names = np.array(row_index_names))
        except: 
            row_index_named = None
        # combine into a dataframe
        df = pd.DataFrame(data = np.array(pivot_values), 
                          index = row_index_named, 
                          columns = column_index).astype('float')
    except:
        df = pd.DataFrame()
    return df

def format_report(response):
    summary = format_summary(response)
    pivot = format_pivot(response)
    if pivot.columns.nlevels == 2:
        summary.columns = [['']*len(summary.columns), summary.columns]
    
    return(pd.concat([summary, pivot], axis = 1))

def run_report(body, credentials_file):
    #Create service credentials
    credentials = service_account.Credentials.from_service_account_file(credentials_file, 
                                scopes = ['https://www.googleapis.com/auth/analytics.readonly'])
    #Create a service object
    service = build('analyticsreporting', 'v4', credentials=credentials)
    
    #Get GA data
    response = service.reports().batchGet(body=body).execute()
    
    return(format_report(response))
```

```python
your_view_id = '####'
ga_keys = 'user_key.json'
```

```python 
body = {'reportRequests': [{'viewId': your_view_id, 
                            'dateRanges': [{'startDate': '2021-01-01', 'endDate': '2021-04-30'}],
                            'metrics': [{'expression': 'ga:users'}, 
                                        {"expression": "ga:bounceRate"}],
                            'dimensions': [{'name': 'ga:yearMonth'}],
                            "pivots": [{"dimensions": [{"name": "ga:channelGrouping"}],
                                        "metrics": [{"expression": "ga:users"},
                                                    {"expression": "ga:bounceRate"}]
                                       }]
                          }]}
```

```python 
summary_body = {'reportRequests': [{'viewId': your_view_id, 
                            'dateRanges': [{'startDate': '2021-01-01', 'endDate': '2021-02-28'}],
                            'metrics': [{'expression': 'ga:sessions'}, 
                                        {'expression': 'ga:totalEvents'}, 
                                        {"expression": "ga:avgSessionDuration"}],
                            'dimensions': [{'name': 'ga:country'}],
                          }]}

```

```python 
pivot_body = {'reportRequests': [{'viewId': your_view_id, 
                            'dateRanges': [{'startDate': '2021-01-01', 'endDate': '2021-02-28'}],
                            'dimensions': [{'name':  "ga:channelGrouping"}],
                            "pivots": [{"dimensions": [{"name": 'ga:yearMonth'}],
                                        "metrics": [{"expression": "ga:users"},
                                                    {"expression": "ga:newUsers"},
                                                    {"expression": "ga:timeOnPage"}]
                                       }]
                          }]}

```


```python 
short_body = {  "reportRequests":
  [{
      "viewId": your_view_id,
      "dateRanges": [{"startDate": "7daysAgo", "endDate": "yesterday"}],
      "metrics": [{"expression": "ga:users"}]
    }]}

```


```python 
untidy_body = {'reportRequests': [{'viewId': your_view_id, 
                            'dateRanges': [{'startDate': '2021-01-01', 'endDate': '2021-02-28'}],
                            "pivots": [{"dimensions": [{"name": 'ga:yearMonth'}, {"name": "ga:channelGrouping"}],
                                        "metrics": [{"expression": "ga:users"},
                                                    {"expression": "ga:timeOnPage"}]
                                       }]
                          }]}

```

## This is the block I run that produces the error: 

```python 
ga_report = run_report(body, ga_keys)
ga_report
```

```python 
HttpError                                 Traceback (most recent call last)
/var/folders/kl/qjwkkx690z5240hbdl9fq7fh0000gn/T/ipykernel_34728/1702095285.py in <module>
----> 1 ga_report = run_report(body, ga_keys)
      2 ga_report

/var/folders/kl/qjwkkx690z5240hbdl9fq7fh0000gn/T/ipykernel_34728/4044629936.py in run_report(body, credentials_file)
     75 
     76     #Get GA data
---> 77     response = service.reports().batchGet(body=body).execute()
     78 
     79     return(format_report(response))

~/opt/anaconda3/lib/python3.9/site-packages/googleapiclient/_helpers.py in positional_wrapper(*args, **kwargs)
    128                 elif positional_parameters_enforcement == POSITIONAL_WARNING:
    129                     logger.warning(message)
--> 130             return wrapped(*args, **kwargs)
    131 
    132         return positional_wrapper

~/opt/anaconda3/lib/python3.9/site-packages/googleapiclient/http.py in execute(self, http, num_retries)
    936             callback(resp)
    937         if resp.status >= 300:
--> 938             raise HttpError(resp, content, uri=self.uri)
    939         return self.postproc(resp, content)
    940 

HttpError: <HttpError 403 when requesting https://analyticsreporting.googleapis.com/v4/reports:batchGet?alt=json returned "User does not have sufficient permissions for this profile.". Details: "User does not have sufficient permissions for this profile.">
```
