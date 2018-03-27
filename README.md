# Python Google Translation Service SSE for Qlik Sense

## REQUIREMENTS

- **Assuming prerequisite: [Python with Qlik Sense AAI – Environment Setup](https://www.dropbox.com/s/dhmd3vm7oqurn2m/DPI%20-%20Qlik%20Sense%20AAI%20and%20Python%20Environment%20Setup.pdf?dl=0)**
	- This is not mandatory and is intended for those who are not as familiar with Python to setup a virtual environment. Feel free to follow the below instructions flexibly if you have experience.
- Qlik Sense February 2018+
- Python 3.5.3 64 bit
- Python Libraries: grpcio, googletrans, hyper

## LAYOUT

- [Prepare your Project Directory](#prepare-your-project-directory)
- [Install Python Libraries and Required Software](#install-python-libraries-and-required-software)
- [Setup an AAI Connection in the QMC](#setup-an-aai-connection-in-the-qmc)
- [Copy the Package Contents and Import Examples](#copy-the-package-contents-and-import-examples)
- [Prepare And Start Services](#prepare-and-start-services)
- [Leverage Python Translation Analysis from within Qlik Sense](#leverage-python-translation-analysis-from-within-qlik-sense)

 
## PREPARE YOUR PROJECT DIRECTORY
>### <span style="color:red">ALERT</span>
><span style="color:red">
>Virtual environments are not necessary, but are frequently considered a best practice when handling multiple Python projects.
></span>

1. Open a command prompt
2. Make a new project folder called QlikSenseAAI, where all of our projects will live that leverage the QlikSenseAAI virtual environment that we’ve created. Let’s place it under ‘C:\Users\{Your Username}’. If you have already created this folder in another guide, simply skip this step.
3.
3. We now want to leverage our virtual environment. If you are not already in your environment, enter it by executing:

```shell
$ workon QlikSenseAAI
```

4. Now, ensuring you are in the ‘QlikSenseAAI’ folder that you created (if you have followed another guide, it might redirect you to a prior working directory if you've set a default, execute the following commands to create and navigate into your project’s folder structure:
```
$ cd QlikSenseAAI
$ mkdir Translate
$ cd Translate
```


5. Optionally, you can bind the current working directory as the virtual environment’s default. Execute (Note the period!):
```shell
$ setprojectdir .
```
6. We have now set the stage for our Sentiment environment. To navigate back into this project in the future, simply execute:
```shell
$ workon QlikSenseAAI
```

This will take you back into the environment with the default directory that we set above. To change the
directory for future projects within the same environment, change your directory to the desired path and reset
the working directory with ‘setprojectdir .’


## INSTALL PYTHON LIBRARIES AND REQUIRED SOFTWARE

1. Open a command prompt or continue in your current command prompt, ensuring that you are currently within the virtual environment—you will see (QlikSenseAAI) preceding the directory if so. If you are not, execute:
```shell
$ workon QlikSenseAAI
```
2. Execute the following commands. If you have followed a previous guide, you have more than likely already installed grpcio):

```shell
$ pip install grpcio
$ pip install googletrans
$ pip install hyper
```

## SET UP AN AAI CONNECTION IN THE QMC

1. Navigate to the QMC and select ‘Analytic connections’
2. Fill in the **Name**, **Host**, and **Port** parameters -- these are mandatory.
    - **Name** is the alias for the analytic connection. For the example qvf to work without modifications, name it 'PythonTranslate'
    - **Host** is the location of where the service is running. If you installed this locally, you can use 'localhost'
    - **Port** is the target port in which the service is running. This module is setup to run on 50091, however that can be easily modified by searching for ‘-port’ in the ‘\_\_main\_\_.py’ file and changing the ‘default’ parameter to an available port.
3. Click ‘Apply’, and you’ve now created a new analytics connection.


## COPY THE PACKAGE CONTENTS AND IMPORT EXAMPLES

1. Now we want to setup our translation service and app. Let’s start by copying over the contents of the example
    from this package to the ‘..\QlikSenseAAI\Translate\’ location. Alternatively you can simply clone the repository.
2. After copying over the contents, go ahead and import the example qvf found [here](https://www.dropbox.com/s/9fha0xqm9c3bnuv/DPI%20-%20Python%20Translation.qvf?dl=0).
3. Lastly, import the qsvariable extension zip file found here using the QMC.


## PREPARE AND START SERVICES

1. At this point the setup is complete, and we now need to start the translation extension service. To do so, navigate back to the command prompt. Please make sure that you are inside of the virtual environment.
2. Once at the command prompt and within your environment, execute (note two underscores on each side):
```shell
$ python __main__.py
```
3. We now need to restart the Qlik Sense engine service so that it can register the new AAI service. To do so,
    navigate to windows Services and restart the ‘Qlik Sense Engine Service’
4. You should now see in the command prompt that the Qlik Sense Engine has registered the functions Translate
    and TranslateScript from the extension service over port 50091, or whichever port you’ve chosen to leverage.


## LEVERAGE PYTHON TRANSLATION ANALYSIS FROM WITHIN SENSE

1. The *Translate()* function leverages the [Googletrans package](http://py-googletrans.readthedocs.io/en/latest/) and accepts three mandatory arguments:
    - *Text (string)*: i.e. a sentence, paragraph, etc
    - *Source (string)*: the language code of the source (see below)
    - *Destination (string)*: the language code of the destination (see below)
2. Example function calls:
	
    *Translate text from English to Japense*:
    ``` PythonTranslate.Translate(text,’en’,’ja’) ``` 
    
    *Translate text from German to English*:
    ``` PythonTranslate.Translate(text,’de’,’en’) ```
3. There is another script function exposed called *TranslateScript()*. This function can only be used in the script and is leveraged via the [**LOAD ... EXTENSION ...**](https://help.qlik.com/en-US/sense/February2018/Subsystems/Hub/Content/Scripting/ScriptRegularStatements/Load.htm) mechanism which was added as of the February 2018 release of Qlik Sense. This function takes four fields:
       
       text, a numeric id, source language, destination language

See the below example of how to utilize the *TranslateScript()* function in the load script:

```
Data:
FIRST 10
LOAD
    business_id,
    "date",
    RecNo() AS review_id,
    stars,
    "text",
    "type",
    user_id,
    cool,
    useful,
    funny,
    'en' AS English,
    'ja' AS Japanese,
    'tr' AS Turkish,
    'sv' AS Swedish,
    'ru' AS Russian,
    'fr' AS French,
    'ar' AS Arabic
FROM [lib://Builds (qlik_qservice)/TranslationData\yelp.csv]
(txt, utf8, embedded labels, delimiter is ',', msq);


// PYTHON
Japanese:
LOAD
	Field1 AS review_id,
	Field2 AS "Translated Text (Japanese)"
EXTENSION PythonTranslate.TranslateScript(Data{text,"review_id",English,Japanese});

Turkish:
LOAD
	Field1 AS review_id,
	Field2 AS "Translated Text (Turkish)"
EXTENSION PythonTranslate.TranslateScript(Data{text,"review_id",English,Turkish});

DROP FIELDS English, Japanese, Turkish;
```

**_The core concept is that translation can be done in the script and physically loaded into the model, or done dynamically on the front-end. You could implement section access so that a user only has access to a single language code, and then have that language code automatically fill in the “destination” argument of the function, thereby applying custom real-time translation per user without any data loaded._**


The two-letter codes below are what are accepted as the “source” and “destination” arguments for both Python functions"

```python
LANGUAGES = {
'af': 'afrikaans',
'sq': 'albanian',
'am': 'amharic',
'ar': 'arabic',
'hy': 'armenian',
'az': 'azerbaijani',
'eu': 'basque',
'be': 'belarusian',
'bn': 'bengali',
'bs': 'bosnian',
'bg': 'bulgarian',
'ca': 'catalan',
'ceb': 'cebuano',
'ny': 'chichewa',
'zh-cn': 'chinese (simplified)',
'zh-tw': 'chinese (traditional)',
'co': 'corsican',
'hr': 'croatian',
'cs': 'czech',
'da': 'danish',
'nl': 'dutch',
'en': 'english',
'eo': 'esperanto',
'et': 'estonian',
'tl': 'filipino',
'fi': 'finnish',
'fr': 'french',
'fy': 'frisian',
'gl': 'galician',
'ka': 'georgian',
'de': 'german',
'el': 'greek',
'gu': 'gujarati',
'ht': 'haitian creole',
'ha': 'hausa',
'haw': 'hawaiian',
'iw': 'hebrew',
'hi': 'hindi',
'hmn': 'hmong',
'hu': 'hungarian',
'is': 'icelandic',
'ig': 'igbo',
'id': 'indonesian',
'ga': 'irish',
'it': 'italian',
'ja': 'japanese',
'jw': 'javanese',
'kn': 'kannada',
'kk': 'kazakh',
'km': 'khmer',
'ko': 'korean',
'ku': 'kurdish (kurmanji)',
'ky': 'kyrgyz',
'lo': 'lao',
'la': 'latin',
'lv': 'latvian',
'lt': 'lithuanian',
'lb': 'luxembourgish',
'mk': 'macedonian',
'mg': 'malagasy',
'ms': 'malay',
'ml': 'malayalam',
'mt': 'maltese',
'mi': 'maori',
'mr': 'marathi',
'mn': 'mongolian',
'my': 'myanmar (burmese)',
'ne': 'nepali',
'no': 'norwegian',
'ps': 'pashto',
'fa': 'persian',
'pl': 'polish',
'pt': 'portuguese',
'pa': 'punjabi',
'ro': 'romanian',
'ru': 'russian',
'sm': 'samoan',
'gd': 'scots gaelic',
'sr': 'serbian',
'st': 'sesotho',
'sn': 'shona',
'sd': 'sindhi',
'si': 'sinhala',
'sk': 'slovak',
'sl': 'slovenian',
'so': 'somali',
'es': 'spanish',
'su': 'sundanese',
'sw': 'swahili',
'sv': 'swedish',
'tg': 'tajik',
'ta': 'tamil',
'te': 'telugu',
'th': 'thai',
'tr': 'turkish',
'uk': 'ukrainian',
'ur': 'urdu',
'uz': 'uzbek',
'vi': 'vietnamese',
'cy': 'welsh',
'xh': 'xhosa',
'yi': 'yiddish',
'yo': 'yoruba',
'zu': 'zulu'
}
```
