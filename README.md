# ATTENTION

This project dates back to 2/26/2017 when I was working both as Rater and Auditor at [Appen Butlerhill](https://appen.com/),  it is not complete within its scope of functionalities and will not be maintained anymore, some parts have been deleted due to client-independent contractor confidentiality agreement.

# Greetings fellow visitor!

The very main purpose of the following content is to build a model to predict labels in a review/evaluation/job, we will take advantage of Selenium Webdriver to find, extract and store the data provided by the [Client](https://www.facebook.com/pg/facebook/about/), later we will explore this data, clean it, fill into several training models and aim for the highest accuracy possible.




## About the Author
<img style="float: left; margin-right:10px; margin-left:10px" src="https://scontent.frec5-1.fna.fbcdn.net/v/t1.0-9/15941393_1501633696543698_5959706666543527839_n.jpg?oh=1fc3b1e567c71bdad8cc14e79a4f1392&oe=5973DDA6">

**Eduardo Gordilho** ([Facebook](https://www.facebook.com/edugordilho), [Linkedin](https://www.linkedin.com/in/eduardo-gordilho-99193550/), [GitHub](https://github.com/eduord)),
I'm a Civil Engineer with a B.Eng. (5 yrs.) at [UFPE](https://www.ufpe.br/) ([QS Rank](https://www.topuniversities.com/universities/universidade-federal-de-pernambuco-ufpe)) with initial roots on Electric-Electronic Engineering.

Recently I've been very interested in data analysis and machine learning, I don't code for living and I don't master any aspects of what's being presented below, but hopefully you'll find this practice project interesting.

**<div style="text-align: right">Hope you enjoy it,
<br>
Eduardo.</div>**

---

Basic functionality:

- Go inside [SRT](https://our.intern.facebook.com/intern/search_team)
- Set up your project, save and enqueue and start a rating session.
- For each result it'll:
   - Check for errors
       - Handle errors
   - Identify the module
       - Gather the data
       - Produce DataFrames
       - Store DataFrames into local files
       

# Content Summary

This section will make your navigation through this notebook easier, come back here by CTRL + F (Windows) and type "Summary".

## [Setting Up](#setting_up)
- [Importing Dependencies](#dependencies)
- **Possibily Optional**: [Setting Up Cookies](#setting_up) (If it's your 1st time here)
    - [Saving Browser Session](#saving_browser_session)
- [Start a new session](#start_a_new_session)

## [Exploring Functions](#exploring_functions)
- [Function: Get all element attributes](#fun_get_all_element_attributes)
- [Function: Get all butons as List](#fun_get_all_butons_as_list)
- [Function: Get all butons as Dictionary](#fun_get_all_buttons_as_dict)

**NOTE**: This is only a section, DOES NOT include all functions contained in this document.

## [User Interaction (UI)](#user_interaction)
- [Click Button](#click_button)
    - [Function: Click Button by String identifier](#fun_click_button)
    - [Function: Go to SRT Main Screen](#fun_goto_main_srt_screen)
- [Navigating through jobs](#wrapper_fun_jobs_navigation)
    - [Goto next job](#fun_next_job)
    - [Goto previous job](#fun_previous_job)
- [Set Current Experiment by String identifier](#fun_set_current_experiment_to)
- [Save and Enqueue](#fun_save_and_enqueue)
- [See more content](#fun_see_more)
- [Opens a new tab](#fun_open_tab)
- [Close error pop](#fun_close_error_popup)

## [Detection](#detection)
- [Identifying Screen Modules and Errors](#wrapper_identifying)
    - [Function: Status](#fun_status)
    - [UNFINISHED SECTION: Errors](#errors)

## [Data Extraction](#data_extraction):
- [Function: Get Main Screen as dict](#fun_get_main_screen_as_dict)
- [Function: Get Today Queries as DataFrame](#fun_get_today_queries_info_as_dataframe)
- [Function: Get Today Queries as List of Dictionaries](#fun_geT_today_queries_info_as_list_of_dicts)
- [Get Jobs info](#wrapper_fun_get_job_info)
    - [Function: Get job info as dict](#fun_get_job_info_as_dict)
    
## [Definitions](#definitions)
- [Definition: Status list](#def_status_list)
- [Definition: Modules list](#def_modules_list)

## [Wrangling](#wrangle)
- [Basic Wrangle](#basic_wrangle)
    - [Function: UTime convertion](#fun_utime_conversion)
    - [Function: Get node text excluding children](#fun_get_text_excluding_children)

- [Modules:](#modules_wrangle)
    - [Wrangle Page and Group module](#module_page_and_group)
	- [Wrangle Video](#module_video)
	- [Wrangle Profile](#module_profile)
		- [Definition: Triggers](#def_triggers)
    - [Wrangle Photo](#module_photo)
    - [Wrangle External Contentl](#module_external)
    - [Wrangle User Content](#module_user_content)

---

# <a name="dependencies">Importing Dependencies</a>


```python
# Our main source of power
from selenium import webdriver
# Allow us to perform hotkey interactions
from selenium.webdriver.common.keys import Keys
# Is used to hover mouse function
from selenium.webdriver.common.action_chains import ActionChains

# Vectorized operations and display multiple DataFrames on screen
import pandas as pd
import numpy as np
from IPython.core import display as ICD

# Helps with default dict structures
import collections
# Helps us with time data manipulation
import datetime
# Track performace when needed
import time
# To import our cookies and in the future the classifiers
import pickle

# To help parse selenium image to Pillow
import io
# To visualize and manipulate images
from PIL import Image
from PIL import ImageGrab
import matplotlib.pyplot as plt
%matplotlib inline
# To parse profile pictures links to Pillow
import urllib

# To help with file storage
import os
# To help us with waiting page to load functions
from time import sleep

# To help with string manipulation
from nltk.tokenize import word_tokenize
import re
from unidecode import unidecode
# Remove double letters
from itertools import groupby
# Secret
import random
```

---

# <a name="setting_up">Setting up</a>

We will use your cookies to avoid 2-step-verification and additional initial code to log into your account.

Therefore the following function we'll be used to help us with that.


```python
def save_cookies():
    pickle.dump(browser.get_cookies() , open("SRTCookies.pkl","wb"))
```

## <a name="saving_browser_session">Saving Browser Session</a>

The following chunk of code is inteded **if this is your first time** through this code.

**ATTENTION**: The chunk bellow was purposefully modified to *Raw NBConvert* to avoid unintentional execution. If you need to go through this step please change the chunk type to *Code*:

By either doing this with the cursor:

![Convert Raw NBConvert Chunk to Code Chunk](chunk_to_code.png)

Or by clicking in the Chunk and then pressing the key **[ESC]** (note that the vertical line that once was green must now be blue) followed by pressing the key for letter **[M]**.
driver = webdriver.Chrome()
driver.get('https://our.intern.facebook.com/intern/search_team')
save_cookies()
## <a name="start_a_new_session">Start a New Session</a>


```python
def start_new_session():
    session = webdriver.Chrome(executable_path="./chromedriver.exe")
    # Open any website to allow us to load cookies
    session.get('https://www.facebook.com/')
    # Load cookies
    for cookie in pickle.load(open("SRTCookies.pkl", "rb")):
        session.add_cookie(cookie)
    return(session)
```

---

# <a name="exploring_functions">Exploring Functions</a>

This section will support basic evaluations, functions here are for the purpose of evaluating some conditions with **WebElements** and **Buttons**.

<a name="fun_all_element_attributes"></a>


```python
# Javascript func, not Python
def get_all_element_attributes(element):
    return(browser.execute_script('var items = {}; for (index = 0; index < arguments[0].attributes.length; ++index) { items[arguments[0].attributes[index].name] = arguments[0].attributes[index].value }; return items;', element))
```

<a name="fun_get_all_butons_as_list"></a>


```python
# List with button elements (text,css_xpath,index)
def get_all_buttons_as_list():
    buttons_list = []
        
    xpath = '//input[@type="button"]'
    [buttons_list.append((e.get_attribute("value"),xpath,i)) for i,e in enumerate(browser.find_elements_by_xpath(xpath))]

    xpath = '//a[@role="button" and not(contains(@style,"display:none"))]'
    [buttons_list.append((e.text,xpath,i)) for i,e in enumerate(browser.find_elements_by_xpath(xpath))]

    xpath = '//button'
    [buttons_list.append((e.text,xpath,i)) for i,e in enumerate(browser.find_elements_by_xpath(xpath))]
    return(buttons_list)
```

<a name="fun_get_all_buttons_as_dict"></a>


```python
def get_all_buttons_as_dict():
    buttons_dict = {}
    for e in get_all_buttons_as_list():
        buttons_dict[e[0]] = {'xpath': e[1],'index':e[2]}
    return buttons_dict
```


```python
def find_element_from_dict(parent, dictionary,first_level_only = False, precise_class = False):
    
    # If dictionary has information about find parameters, use it and ignore what the user is giving as parameters
    if 'find_parameters' in dictionary.keys():
        parameters = dictionary['find_parameters']
        first_level_only = parameters['first_level_only']
        precise_class = parameters['precise_class']    

    if first_level_only:
        depth = '/'
    else:
        depth = '//'
    
    if 'class' in dictionary.keys():
        if precise_class:
            xpath = '.{}{}[@class="{}"]'.format(depth,dictionary['tag'],dictionary['class'])  
        else:
            xpath = '.{}{}[contains(@class,"{}")]'.format(depth,dictionary['tag'],dictionary['class'])
    else:
        xpath = '.{}{}'.format(depth,dictionary['tag'])
        result = parent.find_element_by_xpath(xpath)

    return(parent.find_element_by_xpath(xpath))
```


```python
def find_element(parent, tag, class_ = None, first_level_only = False, precise_class = False):
    if first_level_only:
        depth = '/'
    else:
        depth = '//'

    if class_:
        try:
            if precise_class:
                return(parent.find_element_by_xpath('.{}{}[@class="{}"]'.format(depth,tag,class_)))
            else:
                return(parent.find_element_by_xpath('.{}{}[contains(@class,"{}")]'.format(depth,tag,class_)))
        except:
            pass
    else:
        try:
            return(parent.find_element_by_xpath('.{}{}'.format(depth,tag)))
        except:
            pass
```

---

# <a name="user_interaction">User Interaction</a>

## <a name="click_button"> Click button</a>

Here we'll define a [click function](#fun_click_button) that will receive a string as input and will attempt to click an element that has that string parsed.

If you have problems with this function please check the [*function to get all buttons as list*](#fun_get_all_butons_as_list) and see how the elements are gathered, you may have to change how it gathers e[0] (the text part).

<a name="fun_click_button"></a>


```python
def click_button(string_identifier):
    for e in get_all_buttons_as_list():
        # e[0] - Text
        # e[1] - Selector as String
        # e[2] - Element index in that browser.find_elements_by_css_selector(css_class_selector)
        if string_identifier in e[0]:
            # Print Button Clicked
            print(e)
            browser.find_elements_by_xpath(e[1])[e[2]].click()
            return(True)
    print("Couldn't find button, if you have specified the string correctly please check the function get_all_buttons_as_list() and look for missing button types detection")
    return(False)
```

<a name="fun_goto_main_srt_screen"></a>


```python
def goto_main_srt_screen():
    srt_links = ["https://our.intern.facebook.com/intern/search_team","https://intern.facebook.com/intern/search_team", "https://our.cstools.facebook.com/intern/search_team"]
    while status()['screen'] != 'srt_main':
        time_waited = 0
        maxtime = 10
        if browser.current_url not in srt_links:
            print("It seems like you're not on SRT")
            print("Your current url is: {}".format(browser.current_url))
            print("Not contained in a predefined list of SRT mirrors")
            print("Please make sure to update the mirror to srt_links variable if {} is a SRT Mirror".format(browser.current_url))
            print("Redirecting to Main SRT Screen:")
            browser.get("https://our.intern.facebook.com/intern/search_team")
            print("Done!")
            sleep(1)
            time_waited += 1
            if time_waited > maxtime:
                browser.get("https://our.intern.facebook.com/intern/search_team")
                print("Trying again")
        else:
            return("We're already on the Main SRT Screen")
    return("Successfully loaded main SRT Page")
```

<a name="wrapper_fun_jobs_navigation"></a>
<a name="fun_next_job"></a>
<a name="fun_previous_job"></a>


```python
def next_job():
    return(browser.find_element_by_xpath('//body[contains(@class,"UIInternPage")]').send_keys(Keys.ARROW_RIGHT))
def previous_job():
    return(browser.find_element_by_xpath('//body[contains(@class,"UIInternPage")]').send_keys(Keys.ARROW_LEFT))
```

<a name="fun_set_current_experiment_to"></a>


```python
# Receives an string identifier for the project and set the project as current experiment
def set_current_experiment_to(string_identifier):
    if 'srt_main' in status()['screen']:
        # Accessing Experiments Dropdown List Current Text
        experiments_button = browser.find_element_by_css_selector("._42ft._4jy0._55pi._5vto._55_p._2agf._p._4jy3._517h._51sy")
        if string_identifier in experiments_button.text:
            print("Seems like we're already evaluating the right experiment: {}".format(experiments_button.text))
        else:
            current_project = experiments_button.text
            while not(browser.find_elements_by_xpath('//div[contains(@class,"_6a _6b") and contains(@class,"openToggler")]')):
                experiments_button.click()
                sleep(1)
            # CSS Selector that contains each available experiment
            experiments = browser.find_elements_by_css_selector("._54nh") # ._54nh
            experiments_list = []
            for i,e in enumerate(experiments):
                experiments_list.append([i,e.text])
            indice = [i for i, s in enumerate(experiments_list) if string_identifier in s[1]]
            if len(indice) > 1:
                print('Seems like we have more than one of your experiments identified by "{}", they are:'.format(string_identifier))
                for i in indice:
                    print("{}".format(experiments_list[i][1]))
                print("Please specify accordingly")
            else:
                experiments[indice[0]].click()
                print("Project was: {} \n\
                Set to: {}".format(current_project,experiments_button.text))
```

<a name="fun_save_and_enqueue"></a>


```python
def save_and_enqueue():
    # Getting all text from buttons in one line
    if 'Clear Assignment' in [e[0] for e in get_all_buttons_as_list()]:
        print("It seems like you have already enqueued")
    else:
        click_button("Save and Enqueue")
```

<a name="fun_see_more"></a>


```python
def see_more():
    try:
        browser.find_element_by_css_selector(".see_more_link_inner").click()
        return(1)
    except:
        return(0)
```

<a name="fun_open_tab"></a>


```python
def open_tab(link):
    browser.execute_script("window.open('%s');"%link)
```

<a name="fun_close_error_popup"></a>


```python
def close_error_popup():
    if click_button("Close") | click_button("OK"):
        print("Error Popup Closed")
    else:
        print("Couldn't find a way to close the error")
    print("No errors found")
```

---

# <a name="detection">Detection</a>

## <a name="wrapper_identifying">Identifying Current Screen, Errors and Modules</a>

In this section every function to identify in which screen we're at will be defined.

Currently I have mapped the following screens:
- Main SRT Screen
- Rating Screen:
    - Each (known) module type screen
    - Query switch screen
    - Known Errors Screens

**Status** if a function that will retrieve basic informative data about what's on our screen. 

Status covers:
- Is the page Loading?
- Screen type check:
    - Are we on main srt screen?
    - Are we on the rating screen?
        - Is this the switch query screen?
- Modules check:
    - Is this a user content?
    - Is this a Profile
    - Is this external content?
    - Is this an owned page?
    - Is this an unowned page?
    - Is this a group?
    - Is this the photos module
    - Is this the video module
        - IS there a SEE MORE button?

Note that ERROR is the only element that is a dict, you can get the error type by

``[error_indicator['error'] for error_indicator in status()['status']]`` 



<a name="fun_status"></a>


```python
modules = ['user_content']
def status():
    status_dict = {'screen': [], 'status': [], 'module': []}
    status = []
    # Loading?
    try:
        if browser.find_element_by_xpath('//div[@class="mal _52jv"]/img'):
            status.append("loading")
            status_dict['status'].append("loading")
        elif browser.find_element_by_xpath('//td[@class="_51m-"]/img'):
            status.append("loading")
            status_dict['status'].append("loading")
    except:
        pass

    # Main SRT SCreen
    try:
        if "Just Go" in [e[0] for e in get_all_buttons_as_list()]:
            status.append('srt_main')
            status_dict['screen'].append("srt_main")
    except:
        pass
    
    # Check if we're in the rating screen
    try:
        # SRT Experiments wrapper
        if browser.find_element_by_xpath('//*[@id="srt_renderer_wrapper"]'):
            status.append('rating_screen')
            status_dict['screen'].append('rating')
            
    except:
        pass
    
    # Check if is Switch Query screen
    try:
        if browser.find_element_by_xpath('//div[contains(@class,"_3a18")]'):
            status.append("query_switch")
            status_dict['status'].append("query_switch")
    except:
        pass
    
    # Checks for Cannot Load data for job [job_id]
    try:
        if "Cannot load" in browser.find_element_by_xpath('//div[contains(@id,"u_0_3")]').text:
            # Dict alternative
            # error_dict = dict.fromkeys(['error'])
            # error_dict['error'] = browser.find_element_by_xpath('//div[contains(@id,"u_0_3")]').text
            # status_dict['status'].append(error_dict)
            error = 'error: ' + browser.find_element_by_xpath('//div[contains(@id,"u_0_3")]').text
            status_dict['status'].append(error)
    except:
        pass
    
    # User Content
    user_content = ['userContentWrapper',"fbUserContent"]
    
    for identifier in user_content:
        try:
            if browser.find_element_by_xpath('//div[contains(@class,"{}")]'.format(identifier)):
                status.append("user_content")
                status_dict['module'].append("user_content")
        except:
            pass
    
    # Profile
    try:
        # div[contains(@class,"_3u1") profile
        if "EntRegularPersonalUser" in browser.find_element_by_xpath('//a[contains(@class,"_ohe")]').get_attribute("data-testid"):
            status.append("profile")
            status_dict['module'].append("profile")
    except:
        pass
    # External content
    try:
        if browser.find_element_by_xpath('//div[contains(@class,"SearchSimpleSingleReviewRenderer")]//b'):
            status.append("external_content")
            status_dict['module'].append("external content")
    except:
        pass
    # Owned Page
    try:
        if "EntOwnedPage" in browser.find_element_by_class_name("_ohe").get_attribute("data-testid"):
            status.append("owned_page")
            status_dict['module'].append("owned_page")
    except:
        pass
    # Unowned Page
    try:
        if "EntUnownedPage" in browser.find_element_by_class_name("_ohe").get_attribute("data-testid"):
            status.append("unowned_page")
            status_dict['module'].append("unowned_page")
    except:
        pass
    # Group
    try:
        if "EntGroup" in browser.find_element_by_class_name("_ohe").get_attribute("data-testid"):
            status.append("group")
            status_dict['module'].append("group")
    except:
        pass
    # Photo
    try:   
        if browser.find_element_by_css_selector("._5t31"):
            status.append("photo")
            status_dict['module'].append("photo")
    except:
        pass
    # Video
    try:
        if 'Videos' in browser.find_element_by_xpath('//div[@class="_51cm"]//span').text:
            status.append("video")
            status_dict['module'].append("video")
    except:
        pass
   
    # Check if see more button available
    try:
        if browser.find_element_by_css_selector(".see_more_link_inner"):
            status.append('see_more')
            status_dict['status'].append("see_more")
    except:
        pass

    # Error Cannot load data from job
    # Handle Something Went Wrong
    # Click on Close Button
    #browser.find_element_by_xpath("//em[contains(.,'Close')]").click()
    return(status_dict)
```

---

# <a name="data_extraction">Data Exctraction</a>

<a name="fun_get_main_screen_as_dict"></a>


```python
def get_main_screen_summary_as_dict():
    main_summary = {}
    if 'srt_main' in status()['screen']:
        table_head = [e.text for e in browser.find_elements_by_xpath('//thead/tr/th/span/span')]
        table_body = [e.text for e in browser.find_elements_by_xpath("//table[@class='uiDataTable mvl _5tss']/tbody/tr/td")]

        for i,e in enumerate(table_head):
            main_summary[e] = table_body[i]
        return(main_summary)
```

<a name="fun_get_today_queries_info_as_dataframe"></a>


```python
def get_today_queries_info_as_dataframe():
    if 'srt_main' in status()['screen']:
        # Get the index of most complete attributes
        ideal_header_index = np.array([len(e.get_attribute("data-tooltip-content").splitlines()) for i,e in enumerate(browser.find_elements_by_xpath('//div[@class="_4fbq"]//a'))]).argmax()
        # Using class of Today Queries to retrieve a list with its information
        today_queries_info_list = [e.get_attribute("data-tooltip-content").splitlines() for e in browser.find_elements_by_xpath('//div[@class="_4fbq"]//a')]
        headers = [e[:e.find(":")] for i,e in enumerate(today_queries_info_list[ideal_header_index])]
        df = pd.DataFrame(columns=headers)

        for i,e in enumerate(today_queries_info_list):
            local_headers = [element[:element.find(":")] for element in today_queries_info_list[i]]
            for j in range(0,len(e)):
                current_information = e[j][e[j].find(":")+1:].strip()
                df.loc[i,local_headers[j]] = current_information
        return(df)
    else:
        print("Apparently we're not on the main SRT Screen")
        return(0)
```

<a name="fun_get_today_queries_info_as_list_of_dicts"></a>


```python
def get_today_queries_info_as_list_of_dicts():
    if 'srt_main' in status()['screen']:
        today_queries_info_list = [e.get_attribute("data-tooltip-content").splitlines() for e in browser.find_elements_by_xpath('//div[@class="_4fbq"]//a')]

        today_queries_list = []
        for i in range(0,len(today_queries_info_list)):
            local_headers = [e[:e.find(":")] for i,e in enumerate(today_queries_info_list[i])]
            local_information = [e[e.find(":")+1:].strip() for i,e in enumerate(today_queries_info_list[i])]
            today_queries_list.append(dict(zip(local_headers,local_information)))
        return(today_queries_list)
    else:
        print("Apparently we're not on the main SRT Screen")
        return(0)
```

<a name="wrapper_fun_get_job_info"></a>
<a name="fun_get_job_info_as_dict"></a>


```python
def get_job_as_dict():
    try:
        return(dict(zip([e.text for e in browser.find_elements_by_xpath('//div[@class="pas _50f8"]')],
                 [e.text for e in browser.find_elements_by_xpath('//div[@class="pas"]')])))
    except:
        return(0)
```

---

# <a name="definitions">Definitions</a>

<a name="def_status_list"></a>


```python
status_list = ["query_switch","srt_main","loading"]
```

<a name="def_modules_list"></a>


```python
modules_list = ["user_content","profile","external_content","owned_page","unowned_page","group","photo","video"]
```

---

# <a name="wrangle">Wrangle</a>
<a name="wrangling"></a>

## <a name="basic_wrangle">Basic wrangling functions</a>

<a name="fun_utime_conversion"></a>


```python
# Since utime is available this function can be handy
def utime_to_dd_mm_aaaa_hh_mm_ss(unix_timestamp):
    return(
    datetime.datetime.fromtimestamp(
        unix_timestamp
    ).strftime('%d-%m-%Y %H:%M:%S')
)
```

<a name="wrapper_fun_text_without_children">
<a name="fun_get_text_excluding_children">


```python
def get_text_excluding_children(driver, element):
    return driver.execute_script("""
var parent = arguments[0];
var child = parent.firstChild;
var ret = "";
while(child) {
    if (child.nodeType === Node.TEXT_NODE)
        ret += child.textContent;
    child = child.nextSibling;
}
return ret;
""", element) 
```


```python
def var_to_str(string):
    string = string.split(',')
    return([item.replace('\n','').strip() for item in string])
```

---

## <a name="modules_wrangle"> Wrangling Modules</a>

<a name="fun_main"></a>

### <a name="module_page">Wrangle Page and Group module</a>
<a name="module_group"></a>
<a name="module_page_and_group"></a>

<a name="fun_page_module"></a>


```python
def page_module():    
    page_wrapper = browser.find_element_by_xpath('//div[contains(@class,"clearfix _zw")]')
    page_element = page_wrapper.find_element_by_xpath('//a[contains(@class,"_ohe")]')
    pg_module = dict()
    pg_module['link'] = page_element.get_attribute('href')
    pg_module['type'] = page_element.get_attribute('data-testid').replace('serp_result_link#0@','')
    pg_module['page_scaled_image_link'] = page_element.find_element_by_xpath('.//img').get_attribute('src')
    header_info_element = page_wrapper.find_element_by_xpath('.//div[contains(@class,"_zs fwb")]')
    pg_module['title'] = unidecode(header_info_element.find_element_by_xpath('./a').text.strip().lower())
    
    # Subheader
    try:
        subheader = page_wrapper.find_element_by_xpath('.//div[contains(@class,"_pac _dj_")]')
    except:
        print("Odd, apparently there's no subheader")
    
    # Place? Businesss? Athlete? Further entity information
    try:
        if get_text_excluding_children(browser,subheader) != u"":
            pg_module['subheader_text'] = get_text_excluding_children(browser,subheader)
        pg_module['subheader_type'] = subheader.find_element_by_xpath('./span').text #subheader.
        pg_module['subheader_info'] = subheader.find_element_by_xpath('./a').text
    except:
        pass
    
    # Ratings and Reviews information
    try:
        ratings = subheader.find_element_by_xpath('.//div[contains(@class,"mrs _4sk4")]')
        pg_module['rating_stars'] = ratings.find_element_by_xpath('.//div[contains(@class,"_6a _2b87")]').text
        pg_module['#ratings'] = ratings.find_element_by_xpath('.//div[contains(@class,"_6a _2h6i")]').text
    except:
        pass
    
    # Description
    try:
        description = page_wrapper.find_element_by_xpath('.//div[contains(@class,"_946")]')
        # Snippets
        pg_module['description'] = description.find_element_by_xpath('.//div[contains(@class,"_52eh")]').text
    except:
        pass

    # Like = Don't like, Liked = Have liked.
    try: 
        like_info = page_wrapper.find_element_by_xpath('.//button[contains(@class,"PageLikeButton")]').text
        if like_info == u"Liked":
            pg_module['likes'] = True
        elif like_info == u"Like":
            pg_module['likes'] = False
        else:
            print("Something went wrong when evaluating if the user likes the page")
    except:
        pass
    
    # Joined Group?
    try:
        joined_text = page_wrapper.find_element_by_xpath('.//a[contains(@data-bt,"join_group")]').text
        if joined_text == "Joined":
            pg_module['joined'] = True
        elif joined_text == "Join":
            pg_module['joined'] = False
        else:
            print("Something went wrong when attempting to detect relationship between searcher and group")
    except:
        pass
    
    # Friends like group? Snippets Information
    try:
        snippets = dict(zip(['snippet_1','snippet_2'],[unidecode(info.text.lower()).strip() for info in browser.find_elements_by_xpath('//div[contains(@class,"_ajw")]')]))
        snippets_information = list(snippets.values())
        friends_like_this = True in ['other friends like this' in information for information in snippets_information]
        if friends_like_this:
            pg_module['friends_like_page'] = True
        else:
            pg_module['friends_like_page'] = False
        pg_module['snippets_information'] = [snippets_information]
    except:
        pass
    # page_df = pd.DataFrame(pg_module)
    return pd.DataFrame(pg_module,index=['information']).transpose()
```

# Wrangle Group Module

### <a name="module_video">Wrangling Video Module</a>



```python
def video_module():
    video_wrapper = browser.find_element_by_xpath('//div[@class="_14au"]')
    video_container = video_wrapper.find_element_by_xpath('.//div[@class="_2uah"]')
    header_wrapper = video_wrapper.find_element_by_xpath('.//div[@class="_2_l3"]')
    title = header_wrapper.find_element_by_xpath('.//div[@class="_14az"]').text
    try:
        subtitle = header_wrapper.find_element_by_xpath('.//div[contains(@class,"_2_k_")]').text
    except:
        subtitle = np.nan
        pass
    # Chunk_2[0] Identifier, Chunk_2[1] Related entity
    chunk_2_wrapper = video_wrapper.find_element_by_xpath('.//div[@class="_42bz _2uab"]')
    chunk_2 = chunk_2_wrapper.find_elements_by_xpath('.//a')
    author = {chunk_2[0].text.encode('utf-8'): chunk_2[1].text}
    if 'By' in chunk_2[0].text:
        author = chunk_2[1].text
    author_webelement = chunk_2_wrapper.find_element_by_xpath('.//a[@href != "#"]')
    author_link = author_webelement.get_attribute('href')
    author_type = author_webelement.get_attribute('data-testid').replace('serp_result_link#0@','')
    chunk_3 = video_wrapper.find_element_by_xpath('.//div[@class="_42b-"]')
    date = float(chunk_3.find_element_by_xpath('.//abbr["data-utime"]').get_attribute("data-utime"))
    views = get_text_excluding_children(browser,chunk_3.find_element_by_xpath('.//div[@class="fsm fwn fcg"]'))
    video_preview_img = video_container.find_element_by_xpath('.//img').get_attribute('src')
    video_link = video_container.find_element_by_xpath('.//a').get_attribute('href')
    video_df = pd.DataFrame([title,subtitle,author,author_type,author_link,date,views,video_link,video_preview_img],
                 index=['title','subtitle','author','author_type','author_link','date','views','video_link','video_preview_img']).transpose()
    return video_df
```

### <a name="module_profile">Wrangle Profile Module</a>

#### "Magic" triggers

- "Lives in [where]
- "Read [what]
- "Listens to [what]
- "Lives in [where] (City)
    - [where] (State)
        - [where] (Country)
- "In a relationship with [person]
    - since [full_month],[dd],[aaaa]
- "Married to [person_name]
    - since [full_month],[dd],[aaaa]
- "Went to [where]
- "Studies [what] at [where]
- "Studied [what] at [ where]
- "Worked at [where]
- "Works at [where]
- [marital_status] · [gender] 
- 1 mutual friend: [name]
- [integer] mutual friends: [list]
- [integer] followers
Extra (found harder):
- "Former" [proffesion] 
    - at [place]
    
    
The function bellow should return what information the profile you're looking at contains.

<a name="def_triggers"></a>


```python
triggers = {'read': ["Read"],
 'work': ["Works","Worked"],
 'mutual_friends': ["mutual"],
 #'other': ['including','and']
 'academic': ["Studies","Studied"],
 'location': ["Lives", "Went","From"],
 'age': ["years old"],
 'marital_status': ["Married","Engaged","relationship"],
 'followers': ["followers"],
 'musical_taste': ["Listens"],
 'sexual_interest': ["Interested"]}

triggers_values = {
 'gender': ['Male', 'Female'],
 'marital_status': ['Single'],
 'mutual_friends': ['including','and'],}
#'sexual_interest': ['males', 'females'],
```


```python
stop_words = [" at ", " to ", " in "," a ", " and ", " em "]
```


```python
def remove_stop_words(string):
    for stop_word in stop_words:
        try:
            match_index = string.index(stop_word)
            string = string[0:match_index] + string[match_index+len(stop_word):]
        except:
            pass
    return(string)
```

#### Gathering Profile Information
# Raw Entities involved
raw_entities = [(e.text,get_all_element_attributes(e)) for e in browser.find_elements_by_xpath( 
    '//div[contains(@data-bt,"ct")]//*[not(*) and text()]')]

```python
def profile_module_info():
    # Profile Info
    author = {}
    author['entity'] = unidecode(get_text_excluding_children(browser,browser.find_element_by_xpath("//div[contains(@class,'fwb')]/a")).lower()).strip()
    try:
        author['alt_name'] = unidecode(get_text_excluding_children(browser,browser.find_element_by_xpath("//div[contains(@class,'fwb')]/a/span")).lower()).strip()
        replace_filter = ['(',')']
        for flt in replace_filter:
            if flt in author['alt_name']:
                author['alt_name'] = author['alt_name'].replace(flt,'')
    except:
        pass

    author['link'] = [e.get_attribute("href") for e in browser.find_elements_by_xpath
                      ('//div[contains(@class,"clearfix")]//a[contains(@class,"_ohe")]')]

    author['picture'] = [e.get_attribute("src") for e in browser.find_elements_by_xpath
                         ('//div[contains(@class,"clearfix")]//a//img')]
    try:
        entity_type = browser.find_element_by_xpath("//div[contains(@class,'fwb')]/a").get_attribute("data-testid")
        entity_type = entity_type[entity_type.index('@')+1:]
        author['entity_type'] = entity_type
    except:
        pass
    author['indicator'] = 'author'
    
    # Snippet: Hard Part
    snippet = []
    for info_chunk in browser.find_elements_by_xpath('//div[contains(@class,"_52eh")]'):
        words = word_tokenize(get_text_excluding_children(browser,info_chunk))
        # Find single information inside chunk
        try:
            for key in triggers.keys():
                for trigger in triggers[key]:
                    info = {}
                    for word in words:
                        if trigger == word:
                            if key in info.keys():
                                info[key] = list(np.append(info[key],
                                                           remove_stop_words(info_chunk.text.replace(trigger,'')))).strip()
                            else:
                                info[key] = remove_stop_words(info_chunk.text.replace(trigger,'')).strip()
                            if get_text_excluding_children(browser,info_chunk) != u"":
                                info['indicator'] = remove_stop_words(get_text_excluding_children(browser,info_chunk)).lower().strip()

        except:
            pass

        try:
            # If we don't have only a value on the multinfo chunk but also an indicator
            # such as 'years old', we must split it differently to handle the value
            splitted_chunk = [segments.strip() for segments in info_chunk.text.split(u'\xb7')]
            for words in splitted_chunk:
                for key in triggers.keys():
                    for trigger in triggers[key]:
                        info = {}
                        if trigger in words:
                            if key in info.keys():
                                info[key] = list(np.append(info[key],
                                                           remove_stop_words(words.replace(trigger,'')))).strip()
                            else:
                                info[key] = remove_stop_words(words.replace(trigger,'')).strip()
                            if get_text_excluding_children(browser,info_chunk) != u"":
                                info['entity'] = unidecode(author['entity'].lower()).strip()
                                snippet.append(info)
                # If the chunk is a multilevel information and it doesn't provide an indicator
                # Just a single information like: "Single", "Female", this chunk will handle it
                try:
                    for key in triggers_values.keys():
                        for trigger in triggers_values[key]:
                            info = {}
                            if trigger in words:
                                if key in info.keys():
                                    info[key] = list(np.append(info[key],words.replace(trigger,'').strip())).lower()
                                else:
                                    info[key] = trigger.strip().lower()
                                info['entity'] = unidecode(author['entity'].lower()).strip()
                                snippet.append(info)
                except:
                    pass
        except:
            pass

        # Finds link in chunks of info
        try:
            # If there's links
            link_elements = info_chunk.find_elements_by_xpath('.//a')
            for link_element in link_elements:
                info = {}
                info['link'] = link_element.get_attribute('href')
                try:
                    entity_type = link_element.get_attribute("data-testid")
                    entity_type = entity_type[entity_type.index('@')+1:]
                    info['entity_type'] = entity_type
                    info['entity'] = unidecode(get_text_excluding_children(browser,link_element).lower()).strip()
                    info['indicator'] = remove_stop_words(get_text_excluding_children(browser,link_element.find_element_by_xpath('..'))).lower().strip()
                except:
                    pass

        except:
            pass
        snippet.append(info)
    snippet_df = pd.concat([pd.DataFrame(author),pd.DataFrame(snippet)],axis=0).groupby(['link']).last().reset_index()
    return(snippet_df)
```


```python
def profile_subheader():
    # Subheader
    info = []
    subheader = {}
    subheader['information'] = [unidecode(get_text_excluding_children(browser,x).lower()).strip() for x in browser.find_elements_by_xpath('//div[contains(@class,"_pac _dj_")]//*') if get_text_excluding_children(browser,x) != '']
    subheader['indicator'] = [unidecode(get_text_excluding_children(browser,x).lower()).strip() for x in browser.find_elements_by_xpath('//div[contains(@class,"_pac _dj_")]')][0]
    info.append(subheader)
    return(pd.DataFrame(info))
```


```python
def profile_module():
    return(pd.concat([profile_module_info()],axis=0).groupby('entity').first().reset_index())
```

#### Gathering Relationship

Covers:

- Is friends?
- Is follower?
- Has shared?
- Have liked?


```python
def find_relationship(): 
    relationship = {}
    try:
        # Entity
        index = get_text_excluding_children(browser,browser.find_element_by_xpath("//div[contains(@class,'fwb')]/a"))
    except:
        # No entity found
        pass
    
    try:
        relationship_element = browser.find_element_by_xpath('//div[contains(@class,"_51xa _cme")]')
    except:
        #not a profile
        pass
    
    # Check if is friends
    try:
        if browser.find_element_by_css_selector(".FriendButton").text == u'Friends':
            relationship['friends'] = True
        elif browser.find_element_by_css_selector(".FriendButton").text == u'Add Friend':
            relationship['friends'] = False
    except:
        relationship['friends'] = None
    
    # Check if there's a friend request
    try:
        if browser.find_element_by_css_selector(".FriendButton").text == u"Respond to Friend Request":
            relationship['received_friend_request'] = True
        if browser.find_element_by_css_selector(".FriendButton").text == u"Friend Request Sent":
            relationship['sent_friend_request'] = True
    except:
        pass
    
    
    # Check if has mutual friends
    relationship['share_mutual_friends'] = False
    try:
        for element in browser.find_elements_by_xpath('//div[contains(@class,"_52eh")]'):
            if 'mutual friend' in element.text:
                relationship['share_mutual_friends'] = True
        if 'mutual friend' in browser.find_element_by_xpath('//div[contains(@class,"_pac _dj_")]').text:
            relationship['share_mutual_friends'] = True
    except:
        pass
    
    # Checks if there's follow info:
    try:
        following_element = relationship_element.find_element_by_xpath('.//button[contains(@class,"FollowButton")]')
        if "Following" in following_element.text:
            relationship['following'] = True
        elif "Folow" in following_element.text:
             relationship['following'] = False
        else:
            print("Something unexpected happened to the check following function")
    except:
        pass

    # Check if user has shared
    try:
        if 'Share' in browser.find_element_by_class_name('_4qba').get_attribute('data-intl-translation'):
            relationship['shared'] = True
        else:
            relationship['shared'] = False
    except:
        pass
    
    # Check if user have liked page
    try:
        if browser.find_element_by_class_name('_48-k').get_attribute('data-testid') == u'fb-ufi-likelink':
            relationship['liked'] = False
    except:
        pass
    try:
        if browser.find_element_by_class_name('_48-k').get_attribute('data-testid') == u'fb-ufi-unlikelink':
            relationship['liked'] = True
    except:
        pass
    
    try:
        if u'Liked' in browser.find_element_by_class_name('PageLikeButton').text:
            relationship['liked'] = True
    except: 
        pass
    
    try:
        if u'Like' in browser.find_element_by_class_name('PageLikeButton').text:
            relationship['liked'] = False
    except:
        pass
        
    # Check if user have liked content
    try:
        if 'unlike' in browser.find_element_by_xpath('//a[contains(@class,"UFILikeLink")]').get_attribute("data-testid"):
            relationship['liked'] = True
        else:
            relationship['liked'] = False
    except:
        pass
    return(relationship)
```

### <a name="module_photo">Wrangle Photos module</a>

## Picture

### Identifying link of the picture


```python
def photo_module():
    photo = {}
    photo_element = browser.find_element_by_xpath('//a[@class="_5t31"]')
    photo['photo_link'] = photo_element.get_attribute('href')
    scaled_element = photo_element.find_element_by_xpath('.//img')
    photo['scaled_link'] = scaled_element.get_attribute('src')
    photo['alt_text'] = scaled_element.get_attribute('alt')
    #Clicking to see content
    photo_element.click()
    
    # INSIDE THE PHOTO
    
    # Is there a See more button in the content? Better try to click it if so
    see_more()
    
    # main_wrapper = browser.find_element_by_xpath('//body//div[@class="clearfix fbPhotoSnowliftPopup"]')
    # Wrapper containing all photo theater elements
    main_wrapper = browser.find_element_by_xpath('//body//div[@class="fbPhotoSnowliftContainer snowliftPayloadRoot uiContextualLayerParent"]//div[@class="clearfix fbPhotoSnowliftPopup"]')
    # Right part (author, comments, etc.)
    right_POI_wrapper = main_wrapper.find_element_by_xpath('.//div[@class="uiScrollableAreaWrap scrollable"]')
    # Left part (picture)
    left_POI_wrapper = main_wrapper.find_element_by_xpath('.//div[@class="stageWrapper lfloat _ohe"]')

    #post_privacy'//div[@id="fbPhotoSnowliftAudienceSelector"]/div[@class="mbs fbPhotosAudienceContainerNotEditable]/a'
    photo['post_privacy'] = browser.find_element_by_xpath('//div[contains(@class,"PhotosAudienceContainerNotEditable")]/a').get_attribute('data-tooltip-content')
    photo['author_name'] = right_POI_wrapper.find_element_by_xpath('//div[contains(@class,"fbPhotoContributorName")]/a').text
    photo['link_profile_mini_pic'] = browser.find_element_by_xpath('//img[contains(@class,"_s0 _5xib _5sq7 _44ma _rw img")]').get_attribute('src')
    photo['photo_utime'] = browser.find_element_by_tag_name('abbr').get_attribute('data-utime')
    photo['photo_text'] = browser.find_element_by_css_selector(".hasCaption").text
    photo_info_webelement = left_POI_wrapper.find_element_by_xpath('//img[contains(@class,"spotlight")]')
    photo['photo_link'] =  photo_info_webelement.get_attribute('src')
    photo['computer_vision'] = photo_info_webelement.get_attribute('alt')

    photo['photo_text'] = right_POI_wrapper.find_element_by_xpath('.//span[contains(@class,"fbPhotosPhotoCaption")]').text

    # Try to gather Tagged people if there's any
    try:
        photo['tag_list'] = right_POI_wrapper.find_element_by_xpath('.//span[@class="fbPhotoTagList"]/span')
        photo['tag_items'] = tag_list.find_elements_by_xpath('.//a[@class="taggee"]')
        photo['tag_name_n_link_tuples'] = [(items.text,items.get_attribute("href")) for items in tag_items]
    except:
        print("No taggs found")
    
    # Defining Photo Df
    photo_df = pd.DataFrame(photo,index=['photo'])
    
    # Gathering comments
    # First check if there's a "View More Comments" button
    try:
        right_POI_wrapper.find_element_by_xpath('.//a[@class="UFIPagerLink"]').click()
    except:
        print("""Sems like there's no "See More Comments" button""")
        
    # Extracting comments          
    try:
        print('hi')
        comments_element = right_POI_wrapper.find_elements_by_xpath('.//div[@class="UFICommentContentBlock"]')
        comments_authors_element_list = [comments_element.find_element_by_xpath('.//a[contains(@class,"UFICommentActorName")]') for comment in comments_element]
        comments_name = []
        comments_link = []
        print('hi')
        comments_name_and_link = [(comments_name.append(author.text),comments_link.append(author.get_attribute("href"))) for author in comments_authors]
        comments_timestamp_elements = [comments_element.find_element_by_xpath('.//abbr[@class="livetimestamp"]') for comment in comments_element]
        comments_timestamp = [timestamp.get_attribute("data-utime") for timestamp in comments_timestamp_elements]
        comments_text_elements = [comments_element.find_element_by_xpath('.//span[contains(@class,"UFICommentBody")]') for comment in comments_element]
        comments_text = [comments_element.text for comment in comments_text_elements]
        
        # Remember to add extra code here to identify the level of the comment (parent comment and child comments)
        # comments_text_parent = [comment.parent for comment in comments_text_elements]
        comments_df = pd.DataFrame([comments_text,comments_name,comments_link,comments_timestamp],index=['comment','author_name','author_link','utime'])
    except:
        print("No comments found")
    return photo_df,comments_df
```

Assuming content is avaliable

## Wrangling information

Here we will define how to gather:
- Post Privacy information
- Author Name
- Link to the mini picture of the author of the photo post
- Utime of the photo post
- Photo Post:
    - Photo Link
    - Computer vision description
    - Photo Text
    - Tagged People
    - Comments
        - Checks if there's a see more button and click it if there is
        - Author
        - Time
        - Profile link

## What is missing

A function to associate each comment to a parent comment, check for nested comments.
Maybe create a list of comments in comments 
or a list of comments that have comments on it and for each one the collection of comments inside, IDK.


```python
photo_df = {}
```

## Visualizing DataFrame with gathered information

### Close Theater Mode after gathering all information
# Close Photo Module Button
browser.find_element_by_xpath('//a[contains(@data-testid,"snowliftclose")]').click()
### <a name="module_external">Wrangle External Module</a>
status()
### <a name="module_user_content">Wrangling User Content</a>


```python
def user_content():
    try:
        main_wrapper = browser.find_element_by_xpath('//div[contains(@class,"userContentWrapper")]')
    except:
        main_wrapper = browser.find_element_by_xpath('//div[contains(@class,fbUserContent)]')
    # Up: Content part (author name,text, original author name...)
    # Down: Relationship part(like,comment,share)
    up_down_division = main_wrapper.find_elements_by_xpath('.//div')
    content_part = up_down_division[0]
    relationship_part = up_down_division[1]
    uc_df = {}
    # Header description part
    uc_df['header'] = content_part.find_element_by_xpath('.//div[contains(@class,"_52eh")]').text
    # Author Part Header
    try:
        author_element = content_part.find_element_by_xpath('.//div[contains(@class,"_5x46")]//div[contains(@class,"clearfix")]')
    except:
        try:
            author_element = content_part.find_element_by_xpath('.//div[contains(@class,"_5x46")]')
        except:
            print("Something went wrong when trying to identify the author of this user content")
            pass
    uc_df['author_link'] = author_element.find_element_by_xpath('.//a').get_attribute('href')
    uc_df['author_mini_pic'] = author_element.find_element_by_xpath('.//img').get_attribute('src')
    author_title_element = content_part.find_element_by_xpath('.//h5')
   
    try:
        if get_text_excluding_children(browser,author_title_element) != u"":
            uc_df['author_name'] =  unidecode(get_text_excluding_children(browser,author_title_element).lower()).strip()
        else:
            uc_df['author_name'] = unidecode(author_title_element.find_element_by_xpath('.//a').text.lower()).strip()
    except:
        print("Couldn't determine author title/name")
    
    # If there's any author text:
    try:
        author_text_part = content_part.find_element_by_xpath('.//div[contains(@class,"_5pbx userContent")]')
        uc_df['author_text'] = author_text_part.find_element_by_xpath('.//p').text
        # here we can have /div class text_exposed_root where there's a paragraph <p> inside with text
        # if the text is greater than a certain number of characters there'll be a Countinue Reading
            # Continue Reading is inside text_exposed_root in a <span> of class text_exposed_link, you can get the original link
            # in the href
        # Entire text or partial text? (Continue Reading available?)
    except:
        pass
    
    # Post date and privacy information
    chunk_3 = content_part.find_element_by_xpath('.//div[contains(@class,"_5pcp _5lel")]')
    uc_df['time'] = chunk_3.find_element_by_xpath('.//abbr[@data-utime]').get_attribute('data-utime')
    try:
        uc_df['privacy'] = chunk_3.find_element_by_xpath('.//a[contains(@class,"Privacy")]').get_attribute('aria-label')
    except:
        try:
            uc_df['privacy'] = chunk_3.find_element_by_xpath('.//a[@data-tooltip-content]').get_attribute('data-tooltip-content')
        except:
            print("Something went wrong when trying to extract the privacy information")
    
    # Action type
    try:
        author_info = author_element.find_element_by_xpath('.//span[@class="fcg"]')
        uc_df['author_action_type'] = get_text_excluding_children(browser,author_info) # Shared, Added
        action_complement_element = author_element.find_element_by_xpath('.//a')
        action_complement = {}
        action_complement['link'] = action_complement_element.get_attribute('href')
        action_complement['complement'] = action_complement_element.text
        uc_df['author_action_complement'] = [action_complement]
    except:
        pass
        
    # Other entities involved?
    try:
        authors_involved_elements = author_info.find_elements_by_xpath('.//a[contains(@class,"profileLink")]')[1:]
        for author in authors_involved_elements:
            # involved[i]: [name,link]
            authors_involved['title'].append(get_text_excluding_children(browser,author))
            authors_involved['link'].append(author.get_attribute('href'))
        if authors_involved['title'] or authors_involed['link']:
            uc_df['authors_involved'] = authors_involved
        print(10*'-')
        print(uc_df['authors_involved'])
        print(10*'-')
    except:
        pass
    
    # This should probably be re-located inside the previous chunk of code
    shared_content_wrapper = content_part.find_element_by_xpath('.//div[contains(@class,"_3x-2")]')
    try:
        # Type Re-share rom someone
        # Shared Wapper also _3x-2
        shared_content_wrapper = content_part.find_element_by_xpath('.//div[contains(@class,"clearfix mtm")]')
        uc_df['original_author_content_link'] = shared_content_wrapper.find_element_by_xpath('./a').get_attribute('href')
        uc_df['original_author_mini_pic'] = shared_content_wrapper.find_element_by_xpath('.//img').get_attribute('src')
    except:
        pass
    
    try:
        #Type Reshare text with link
        shared_content_wrapper = content_part.find_element_by_xpath('.//div[contains(@class,"_5cq3")]')
        # Content linkfwn fcg
        uc_df['original_author_content_link'] = shared_content_wrapper.find_element_by_xpath('.//a').get_attribute('href')
        uc_df['original_content_image_preview'] = shared_content_wrapper.find_element_by_xpath('.//img').get_attribute('src')
    except:
        pass
            
    try:
        shared_content_wrapper = content_part.find_element_by_xpath('.//div[contains(@class,"clearfix mtm")]')
        uc_df['original_author_content_link'] = shared_content_wrapper.find_element_by_xpath('./a').get_attribute('href')
    except:
        pass
               
    #Video
    try:
        uc_df['shared_content_wrapper'] = content_part.find_element_by_xpath('.//div[contains(@class,"_150c")]')
        uc_df['original_author_content_img'] = shared_content_wrapper.find_element_by_xpath('.//a').get_attribute('src')
    except:
        pass
                    
    # Trying to determine original information if it's a video
    try:
        original_content_part = content_part.find_element_by_xpath('.//div[contains(@class,"mtm")]')
        original_author_element = original_content_part.find_element_by_xpath('.//span[contains(@class,"fwn fcg")]')
        uc_df['original_author_title'] = original_author_element.find_element_by_xpath('//a').text
        uc_df['original_author_link'] = original_author_element.find_element_by_xpath('.//a').get_attribute('href')
    except:
        pass

    # Shared a movie
    try:
        original_author_wrapper = original_content_part.find_element_by_xpath('.//div[contains(@class,"_4sdm _6lh _dcs")]')
        original_author_element = original_author_wrapper.find_elements_by_xpath('.//div[contains(@class,"_6lp")]/*')
        uc_df['original_author_title'] = original_author_element[0].text # find_element_by_xpath('.//a').text
        uc_df['original_author_link_element'] = original_author_element[0].find_element_by_xpath('.//a')
        uc_df['original_author_link'] = original_author_content_link = original_author_link_element.get_attribute('href')
        uc_df['original_author_type'] = original_author_element[1].text     
    except:
        pass
    
    # External Link
    try:
        image_elements = original_content_part.find_elements_by_xpath('.//img')
        images = []
        for element in image_elements:
            image = {}
            image['alt'] = element.get_attribute('alt')
            image['link'] = element.get_attribute('src')
            images.append(image)

        uc_df['shared_img'] = [images]
        uc_df['shared_link'] = original_content_part.find_element_by_xpath('.//div[contains(@class,"_3ekx _29_4")]//a').get_attribute('href')
        info_wrapper = original_content_part.find_element_by_xpath('.//div[contains(@class,"_6m3 _--6")]')
        uc_df['description'] = info_wrapper.find_element_by_xpath('.//div[contains(@class,"_6m7 _3bt9")]').text
        uc_df['shared_website_title'] = info_wrapper.find_element_by_xpath('.//div[contains(@class,"_6lz _6mb ellipsis")]').text
        uc_df['headline'] = info_wrapper.find_element_by_xpath('.//div[contains(@class,"mbs _6m6 _2cnj _5s6c")]//a').text
    except:
        pass
    
    # Blue gray checkmarks
    try:
        #//*[@id="u_6g_b"]
        # RnA = Reputability and Authenticity
        original_author_RnA_element = original_author_wrapper.find_element_by_xpath('.//span[@data-tooltip-content]')
        uc_df['original_author_RnA_text'] = original_author_RnA_element.get_attribute('data-tooltip-content') # Reputability and authenticity 
        uc_df['original_author_info'] =  original_author_wrapper.find_element_by_xpath('.//div[contains(@class,"_8yb")]').text.splitlines()
    except:
        print("Something went wrong when trying to figure out which type of content this author shared")
        pass
                        
    try:
        # Original content privacy info, text and time
        original_content_part = content_part.find_element_by_xpath('.//div[contains(@class,"mtm")]')
        uc_df['original_privacy'] = original_content_part.find_element_by_xpath('.//a[contains(@class,"Privacy")]').get_attribute('aria-label')
        uc_df['original_text'] = original_content_part.find_element_by_xpath('.//div[contains(@class,"mtm _5pco")]').text
        uc_df['original_time'] = content_part.find_element_by_xpath('.//abbr[@data-utime]').get_attribute('data-utime')
    except:
        print("Couldn't find original content privacy or text or utime")

    
    # Location 
    try:
        uc_df['location'] = get_text_excluding_children(browser,original_content_part.find_elements_by_xpath('.//a[@class="_5pcq"]')[1])
        uc_df['location_link'] = original_content_part.find_elements_by_xpath('.//a[@class="_5pcq"]')[1].get_attribute('href')
    except:
        print("Couldn't find location information")

    return(pd.DataFrame(uc_df,index=['information']).transpose())
```

# <a name="srt_monitor">SRT Monitor</a>

## Greetings!

We have finally arrived here, I'm so glad that I made this far, Ugh!

Now let's finally monitor my daily life and store the information, YAY!

# User Interaction (UI)

## Asssigning Skip/Do not skip:
- 0 - Skip - Not Valid:
    - skip(0)

- 1 - Do not skip - valid
    - skip(1)

## Asssigning Relevance:

- 1 - Not Useful/Off-Topic
    - relevancce(1)

- 2 - Reasonable Match
    - relevance(2)

- 3 - Primary Match
    - relevance(3)

## Asssigning Confidence:

- 1 - Not confident:
     - confidence(1)

- 2 - Somewhat confident:
    - confidence(2)

- 3 - Confident:
    - confidence(3)


```python
# Click on submit button
def submit():
    return browser.find_element_by_xpath("//button[contains(text(),'Submit')]").click()

# Choose Skip Category 
def set_skip(label):

    if label == 0:
        # 0 - Skip - not valid
        browser.find_elements_by_css_selector("._5ncw")[0].click()
    elif label == 1:
    # 1 - Do not skip - valid
        browser.find_elements_by_css_selector("._5ncw")[1].click()
    else:
        raise Exception("Label must be boolean: 0 - Skip - not valid OR 1 - Do not skip - valid")

# Choose Relevance Category 
def set_relevance(label):
    if label == 1:
        #1 - Not Useful/Off-Topic
        browser.find_elements_by_css_selector("._5ncw")[2].click()
    elif label == 2:
        # 2 - Reasonable Match
        browser.find_elements_by_css_selector("._5ncw")[3].click()
    elif label == 3:
        # 3 - Primary Match
        browser.find_elements_by_css_selector("._5ncw")[4].click()
    else:
        raise Exception("Label must be integer in range [0,3]: 1 - Not Useful/Off-Topic; 2 - Reasonable Match OR 3 - Primary Match")

# Choose Confidence Category
def set_confidence(label):
    if label == 1:
        # 1 - Not confident
        browser.find_elements_by_css_selector("._5ncw")[5].click()
    elif label ==2:
        # 2 - Somewhat confident
        browser.find_elements_by_css_selector("._5ncw")[6].click()
    elif label ==3:
        # 3 - Confident
        browser.find_elements_by_css_selector("._5ncw")[7].click()
    else:
        raise Exception("Label must be integer in range [0,3]: 1 - Not confident; 2 - Somewhat confident OR 3 - Confident")
```

# Compute Decisions


```python
def current_screen():
    return(Image.open(io.BytesIO(browser.get_screenshot_as_png())).convert('LA'))
```


```python
def get_ratings_image_list():
    # Get Rating Elements
    ratings = browser.find_elements_by_css_selector("._5ncw")

    # Get Rating coordinates (x,y) and size (height,width)
    coordinates = [ratings[i].location for i in range(0,len(ratings))]
    size = [ratings[i].size for i in range(0,len(ratings))]

    # Create a list with ratings images
    ratings_list = []
    for i in range(0,len(ratings)):
        x,y,h,w = coordinates[i]['x'],coordinates[i]['y'],size[i]['height'],size[i]['width']
        ratings_list.append(current_screen().crop(box=(x,y,x+w,y+h)))
    return(ratings_list)
```


```python
def print_all_ratings_from_list(ratings_list):
    # Print Ratings
    for e in ratings_list:
        plt.figure()
        plt.imshow(e)
    return()
```


```python
def is_filled(checkbox_image):
    true_value = 181 # First element which differs in index 86
    false_value = 229 # First element which differs in index 86
    comparison = np.asarray(checkbox_image)
    comparison = comparison.reshape(comparison.size,-1)
    if comparison[86].item() == true_value:
        return(True)
    elif comparison[86].item() == false_value:
        return(False)
    else:
        print("Something went wrong with this function, sorry but a better classifier is needed")
        return(-1)
```


```python
def get_decisions_as_list():
    if get_ratings_image_list():
        decisions = []
        for e in get_ratings_image_list():
            decisions.append(is_filled(e))
        return(decisions)
    else:
        return(-1)
```


```python
def get_decisions_as_dict():
    dictx = {}
    if browser.find_elements_by_css_selector(".pvs"):
        labels = browser.find_elements_by_css_selector(".pvs")
    listx = [e.text for e in labels]
    decisions = get_decisions_as_list()
    for i,e in enumerate(listx):
        dictx[e] = decisions[i]
    return(dictx)
```


```python
def get_decisions_textboxes():
    return([e.text for e in browser.find_elements_by_xpath('//textarea[@placeholder="Enter reason here"]')])
```

# Hidden Content (Hoverables)


```python
def save_current_job(driver):
    if not os.path.exists('jobs'):
        os.makedirs('jobs')
    files = [f for f in os.listdir('./jobs/')]
    index_matches = [i for i, s in enumerate(files) if (get_job_internal_info_as_dict()['Job ID'].encode('utf-8') in s
                                                        and 'labels' not in s)]
    if index_matches:
        matches = [files[i] for i in index_matches]
        dashes_index = [e.find('-') for e in matches]
        matched_index = pd.Series([int(matches[i][dash_index+1:-4]) for i,dash_index in enumerate(dashes_index)])
        filename = get_job_internal_info_as_dict()['Job ID'].encode('utf-8') + '-' + str(matched_index.max() + 1) + '.csv'
        # Check missing files
        for i in range(0,matched_index.max()):
            current_name = get_job_internal_info_as_dict()['Job ID'].encode('utf-8') + '-' + str(i) + '.csv'
            if current_name not in matches:
                missing_files.append(current_name)
    else:
        filename = get_job_internal_info_as_dict()['Job ID'].encode('utf-8') + '-0.csv'
    print('Adopted filename: {}'.format(filename))
    missing_files =[]
    print('Missing files: {}'.format(missing_files))
    # Saving job information
    df = get_all_webelements_information_as_pd_dataframe(driver).drop(['parent','webelement'],axis=1)
    df.to_csv('./jobs/{}'.format(filename), sep=',', encoding='utf-8')
    # Saving Labels
    labels = pd.DataFrame(get_decisions_as_dict().items(),columns=['labels','decision'])
    labels = labels.set_index(labels['labels'])
    del labels['labels']
    labels = labels.transpose()
    labels.to_csv('{}{}{}'.format(".\jobs\\", filename[0:-4],'_labels.csv'))
```

## Likely Misstypes


```python
# Removing Consecutive Double Letters
def remove_consecutive_double_letters(string):
    return(''.join(c for c, _ in groupby(string)))
```


```python
def analyse_matches(result):
    # Not an exact match If friends like the page part
    result_tokenized = word_tokenize(result)
    query_tokenized = word_tokenize(query_info_dict()['query'])
    # Matching elements
    matches = [x for x in query_tokenized if x in result_tokenized]
    missing_elements = []
    if len(matches) != len(query_tokenized):
        # Missing Elements
        missing_elements = query_tokenized.copy()
        for match in matches:
            missing_elements.remove(match)
        # partial group/page name matches  with no social engagement  should  be rated as  OFF-TOPIC. 
        
    return({"matches": matches,'missing_elements': missing_elements})
```


```python
row0 = [str(number % 10) for number in range(1,11)]
row1 = ['q','w','e','r','t','y','u','i','o','p','´','[']
row2 = ['a','s','d','f','g','h','j','k','l','ç','~',']']
row3 = ['z','x','c','v','b','n','m',',','.',';']
abnt = [row0,row1,row2,row3]

likely_misstypes = collections.defaultdict(dict)

for i,row in enumerate(abnt):
    
    # Generating a dictionary with all up-down adjacent letters to misstype
    up_row = down_row = None
    if i > 0:
        up_row = abnt[i-1]
    if i < len(abnt) - 1:
        down_row = abnt[i+1]

    for j,letter in enumerate(row):
        # Generating a dictionary with all left-right adjacent letters to misstype
        left_adjacent = right_adjacent = []
        if j > 0:
            left_adjacent = [row[j-1]]
        if j < len(row) - 1:
            right_adjacent = [row[j+1]]
        
        # Generating a dictionary with all up-down adjacent letters to misstype
        up_adjacent = down_adjacent = []
        if j == 0:
            add = 1
            sub = -1
        else:
            add = 1
            sub = 1
        if up_row:
            if j <= len(up_row) - 1:
                up_adjacent = []
                up_adjacent.append(up_row[j])
                if j + 1 < len(up_row):
                    up_adjacent.append(up_row[j+add])
        if down_row:
            if j <= len(down_row) - 1:
                down_adjacent = []
                down_adjacent.append(down_row[j])
                if j + 1 < len(down_row) - 1:
                    down_adjacent.append(down_row[j-sub])
        
        likely_misstypes[letter] = {'left': left_adjacent,'right': right_adjacent,
                                'up': up_adjacent,'down': down_adjacent}
```


```python
possible_fonetic_equivalence = {'s': ['x','z','ç','ss','sc','xc'],
                                'x': ['s','z','cs','ks','ch'],
                                'z': ['s','x'],
                                'e': ['i'],
                                'i': ['e','y'],
                                'y': ['i'],
                                'r': ['rr'],
                                'ç': ['s','ss','c','sc'],
                                'c': ['k','ç','q','ç'],
                                'k': ['c','q'],
                                'q': ['q','c'],
                                'f': ['ph'],
                                'l': ['ll','u'],
                                'u': ['l'],
                                'll': ['l'],
                                'ph': ['f'],
                                'ss': ['s','ç','sc','xc'],
                                'sc': ['s','ç','ss','xc'],
                                'cs': ['x','ks'],
                                'xc': ['s','ss','sc'],
                                'ks': ['cs','x'],
                                'ch': ['x'],
                                'lo': ['lou'], #ex Lourdes, Lurdes.
                                'lu': ['lou'],
                                'lou': ['lu','lo'],
                                'rr': ['r']}
```
