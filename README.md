This Python script pulls data from Facebook Ads Manager, and organises it within a Google Spreadsheet

The following is the google sheet, where the data will land (click on this link to see an example) - 
https://docs.google.com/spreadsheets/d/1LFghb5B0tEIYScX3WKgRiG1m4E988ffz0X8f_t5jJbM/edit#gid=0

The following is the Python Script:

First, assign values to key variables. This information can be retrieved within the Facebook Graph API explorer - 
https://developers.facebook.com/docs/graph-api/



    Client_1_token = 'EAAR9oZARzKPABAMKxxxxxxxxxxx'
    Client_1_app_id = '126xxxxxxxx'
    Client_1_app_secret = 'efaxxxxxxx'
    Client_1_ad_account = 'act_39515665xxxxxxx'
    Client_1_Row_number_1 = 3
    Client_1_Row_number_2 = 2
    Client_1 = [Client_1_token, Client_1_app_id, Client_1_app_secret, Client_1_ad_account, Client_1_Row_number_1, Client_1_Row_number_2]

    Client_2_token = 'EAAR9oZARzKPABAMKxxxxxxxxxxx'
    Client_2_app_id = '126xxxxxxxx'
    Client_2_app_secret = 'efaxxxxxxx'
    Client_2_ad_account = 'act_39515665xxxxxxx'
    Client_2_Row_number_1 = 3
    Client_2_Row_number_2 = 2
    Client_2 = [Client_2_token, Client_2_app_id, Client_2_app_secret, Client_2_ad_account, Client_2_Row_number_1, Client_2_Row_number_2]



#Importing the required packages

    import pandas as pd
    import numpy as np

#I am printing text throughout the code, such that if something breaks, I can trace it's source more quickly

    print('Access_token, App id, and App secret, now set')
    print('     ')

#Importing the required modules

    from facebook_business.api import FacebookAdsApi
    from facebook_business.adobjects.adaccount import AdAccount
    from facebook_business.adobjects.campaign import Campaign
    from facebook_business.adobjects.adset import AdSet
    #from facebook_business.adobjects.adset import ad
    from facebook_business.adobjects.user import User


#Determining which clients to analyse (using variables from above)

    Clients = [[Client_1], [Client_2]]


#Sorting row Length - part 1 of 3 (this is to ensure the first line of the results will land on the third row of my google sheet)

    added_value_to_row = 3 

#I am now setting up a loop, such that the code below will iterate for each client mentioned within the Clients variable

    for i in Clients:  
        my_access_token = i[0][0]
        my_app_id = i[0][1]
        my_app_secret = i[0][2]
        my_ad_account = i[0][3]
    
#Sorting row length - part 2 of 3
    
        my_first_row_number = i[0][4] + added_value_to_row                             
        my_second_row_number = i[0][5] + added_value_to_row     

#Initialising required packages

        Ad_account = AdAccount(my_ad_account) 
        print('Facebook Business api is now re-installed')
        print('     ')
        FacebookAdsApi.init(my_app_id, my_app_secret, my_access_token)
        print('Facebook api is now initialised')
        print('     ')
        pars = {'date_preset':'lifetime', 'limit':3000}


#Acquiring the full list of campaigns

        all_campaigns = Ad_account.get_campaigns(params=pars) 

        c = []
        for i in all_campaigns:
            c.append(i)
    
        print('the variable c, with various id numbers of all prior campaigns, now complete')

#turning this list into a dataframe, and taking the column id

        c_dataframe = pd.DataFrame(c)
        id_set = c_dataframe['id']      

#now creating a list, with just the values of the id's - to be used for later

        my_list_for_use = []       
        for i in id_set:
           my_list_for_use.append(i)

        print('the list for use is now complete')
        print('        ')
        print('         ')


 #Data from yesterday campaign performance 

        var = 'yesterday'
        f_par = {'date_preset':var, 'level':'adset', 'time_increment':'50', 'fields':['campaign_name', 'spend', 'adset_name', 'ad_name', 'purchase_roas']}

        r_list = []
        e_frame = pd.DataFrame(r_list)
        
        x = 0
        my_list = my_list_for_use[0:30]           #Specifying the number of campaigns that I want              
        for i in my_list:
            f_c_insights = Campaign(i).get_insights(params=f_par)  #fields=fields_campaign
            f_c_modified = f_c_insights[0:10000000]
            f_c_list = [x for x in f_c_modified] #now have my temporary list
            f_c_dataframe = pd.DataFrame(f_c_list)   #have now turned this into a dataframe
            e_frame = e_frame.append(f_c_dataframe)     #adding this to my final dataframe with all required dataframes
            x = x + 1
    
        yester_frame = e_frame

        print('Now have yesterframe')     


#Now to create dataframes, pulling data from the last three days, all the way to the last thirty days. Because the treatment of these elements are the same, I will also operate on a loop

#First - defining my variables

        three_day_details = 'last_3d'
        seven_day_details = 'last_7d'
        fourteen_day_details = 'last_14d'
        thirty_day_details = 'last_30d'


        full_days_list = [three_day_details, seven_day_details, fourteen_day_details, thirty_day_details]


        increasing_value=0
        for i in full_days_list:        #going to iterate through full_days_list

            f_par = {'date_preset':i, 'level':'adset', 'time_increment':'50', 'fields':['campaign_name', 'spend', 'adset_name', 'ad_name', 'frequency',                     'purchase_roas']}
            r_list = []
            k_frame = pd.DataFrame(r_list)
            x = 0
            y_list = my_list_for_use[0:15]                          
            for i in my_list:
                f_c_insights = Campaign(i).get_insights(params=f_par)  
                f_c_modified = f_c_insights[0:10000000]
                f_c_list = [x for x in f_c_modified] #now have my temporary list
                f_c_dataframe = pd.DataFrame(f_c_list)   #have now turned this into a dataframe
                k_frame = k_frame.append(f_c_dataframe)     #adding this to my final dataframe with all required dataframes
                x = x + 1

    
#the roas values do not come out in the correct form. Cleaning is therefore required
 
            
            print('now starting the first part of cleaning')

#Cleaning - First Part

            roas = k_frame['purchase_roas'] 
            roas.fillna(0)
            y = []
            for i in roas:
                y.append(i)
                s = y[0:10000]
                s = [x for x in y]                   #s is roas, in the list form
                s = pd.Series(s).fillna(0).tolist()
                x = 0
            for i in s:
                if s[x] == 0:
                    s[x] = [0]
                x = x + 1    
            print('first part cleaning, complete')
 
 #Cleaning - Second Part
 
            r = []
            e = pd.DataFrame(r)
    
            x = 0
            for i in s:
                q = s[x]
                w = pd.DataFrame(q)     
                e = e.append(w)
                x = x + 1  
            print('second part cleaning, complete')
      
        
 #Cleaning - Third Part
 
            value_column = e['value']
            type(value_column)
            value_column_as_list = value_column.tolist()  #changing this to a list, so I can later add this to my original dataframe!!

            k_frame['x_dayROAS'] = value_column_as_list   #have added the new column new_ROAS

            print('now have final version of three frame')
            print('        ')

        
#As the loop iterates through the values of last 3 days, last 7 days, etc, a dataframe representing the respective results of each will be produced
    
            if increasing_value == 0:
                three_frame_final = k_frame
                three_frame_final = k_frame[['campaign_name', 'adset_name','x_dayROAS']]
                three_frame_final['3_day_ROAS'] = three_frame_final['xdayROAS']
                three_frame_final = three_frame_final.drop(['xdayROAS'], axis=1)
            
            elif increasing_value == 1:
                sev_frame_final = k_frame
                sev_frame_final = sev_frame_final[['campaign_name', 'adset_name','x_dayROAS']]
                sev_frame_final['7_day_ROAS'] = sev_frame_final['3dayROAS']
                sev_frame_final = sev_frame_final.drop(['xdayROAS'], axis=1)
            elif increasing_value == 2:
                fourteen_frame_final = k_frame
                fourteen_frame_final = fourteen_frame_final[['campaign_name', 'adset_name','x_dayROAS']]
                fourteen_frame_final['14_day_ROAS'] = fourteen_frame_final['xdayROAS']
                fourteen_frame_final = fourteen_frame_final.drop(['xdayROAS'], axis=1)
            elif increasing_value == 3:
                thirty_frame_final = k_frame
                thirty_frame_final = thirty_frame_final[['campaign_name', 'adset_name','spend', 'frequency', 'x_dayROAS']]
                thirty_frame_final['30_day_ROAS'] = thirty_frame_final['xdayROAS']
                thirty_frame_final = thirty_frame_final.drop(['xdayROAS'], axis=1)

            increasing_value = increasing_value + 1


        print('all working')

    
    
#Finally, have to prepare the final dataframe that will be sent over to google sheets

#I only want to inlude data campaigns if they have data from yesterday. Campaigns which have no data from yesterday must already have been turned off. Thus, there aren't of any interest for the purposes of an Ad review. 

#My final dataframe will contain data these campaigns, over the last 3 days, 7 days, etc

    
        merged_thirty_four = thirty_frame_final.merge(fourteen_frame_final, left_on=['campaign_name', 'adset_name'], right_on=['campaign_name', 'adset_name'])
        merg_thir_four_sev = merged_thirty_four.merge(sev_frame_final, left_on=['campaign_name', 'adset_name'], right_on=['campaign_name', 'adset_name'])
        merg_thir_four_sev_thir = merg_thir_four_sev.merge(three_frame_final, left_on=['campaign_name', 'adset_name'], right_on=['campaign_name', 'adset_name'])
        merg_thir_four_sev_thir_yest = merg_thir_four_sev_thir.merge(yester_frame, left_on=['campaign_name', 'adset_name'], right_on=['campaign_name', 'adset_name'])


        frame_to_go = merg_thir_four_sev_thir_yest[['campaign_name', 'adset_name', 'spend_x', 'frequency','30_day_ROAS', '14_day_ROAS','7_day_ROAS', '3dayROAS', 'spend_y']]


        frame_to_go = frame_to_go.fillna(0)

        print('now have frame_to_go') 

    
   
#And now to connect my dataframe with google sheets

        print ('        ')

        import pandas as pd
        import gspread
        from oauth2client.service_account import ServiceAccountCredentials
        from pandas.io.json import json_normalize
        scope = ['https://docs.google.com/spreadsheets/d/1LVBUB0huHvW8dC2WsCir3xxxxxxx']
        credentials = ServiceAccountCredentials.from_json_keyfile_name('./cool-benefit-290xxxxxxx', scope)

        gc = gspread.authorize(credentials)
        gc = gspread.service_account(filename='./cool-benefit-290520-30a2cxxxxxxx')
        sh = gc.open_by_key('1LVBUB0huHvW8dC2WsCir3Omaxxxxxxxx')
        worksheet = sh.sheet1

        print ('now have worksheet variable ready')
        print ('      ')


        df_to_send = frame_to_go 
        my_list = df_to_send.values.tolist()
    
 #sorting row length - part 3 (this ensures that Client 2 data will not land on Client 1 data, on the google sheet)
 
        added_value_to_row = added_value_to_row + len(df_to_send)     
    
    
#And now to actually send the data!!
    
        for i in my_list:  #my list has all the data, that I want to put onto the spreadsheet, for any given client
            w = []
            q=i #q will be each particular row, that I want to transfer over to the sheet
            w.append(q[0])  #first, just attaching the first two cols as strings
            w.append(q[1])
            x = 2        #starting at x=2, as I don't want to touch the first two values, which I want to keep as strings
            for m in q:
                if x < len(q):
                    w.append(float(q[x]))     #I am turning each numerical value, from a string into a float
                    x = x + 1
                else:
                    print('   ')
            worksheet.insert_row(w, my_first_row_number)                                 
    
        print ('worksheet is now populated')


#Additional component - sending over the total amount spent for the month, which is also helpful

        var = 'this_month'

        f_par = {'date_preset':var, 'level':'adset', 'time_increment':'50', 'fields':['campaign_name', 'spend', 'adset_name', 'ad_name']}
        r_list = []
        t_frame = pd.DataFrame(r_list)
        x = 0
        my_list = my_list_for_use[0:15]         
        for i in my_list:
            f_c_insights = Campaign(i).get_insights(params=f_par)  #fields=fields_campaign
            f_c_modified = f_c_insights[0:10000000]
            f_c_list = [x for x in f_c_modified] #now have my temporary list
            f_c_dataframe = pd.DataFrame(f_c_list)   #have now turned this into a dataframe
            t_frame = t_frame.append(f_c_dataframe)     #adding this to my final dataframe with all required dataframes
            x = x + 1
  

        total = sum(pd.to_numeric(t_frame['spend']))
        total = sum(pd.to_numeric(t_frame['spend']))
        worksheet.update_cell(my_second_row_number, 10, total)      

        print('Cell updated with total amount spent!')
    



