# Development README

## Install

Can be installed on all operating systems and supports Python version >= 3.7, to install run:

``` bash
pip install -e .[tests]
```

For a `zsh` shell, which is the default shell for the new Macs you will need to escape with `\` the brackets:

``` bash
pip install -e .\[tests\]
```

### Running linters

This code base uses flake8 and mypy to ensure that the format of the code is consistent and contain type hints. The flake8 settings can be found in [.flake8](./.flake8) and the mypy settings within [pyproject.toml](./pyproject.toml). To run these linters:

``` bash
isort pymusas_models model_function_tests model_creation_tests
flake8
mypy
```

## General folder structure

* `/pymusas_models` - contains the code that creates all of the PyMUSAS models.
* `/model_creation_tests`
* `/model_function_tests` - The tests are divided up by language, using each language's [BCP 47 language code](https://www.w3.org/International/articles/language-tags/), and then model (currently we only have one model the `rule based tagger`).
    * `/model_function_tests/fr`
        * `/model_function_tests/fr/test_rule_based_tagger.py`
    * `/model_function_tests/it`
        * `/model_function_tests/it/test_rule_based_tagger.py`
    * other language codes
* `/model_creation_tests/test_create_and_install_models.py` - This creates and installs the models used within `tests` and in doing so tests that this part of the code base works. **Note** that we install the models to a temporary Python virtual environment.

The testing structure of `/model_function_tests` has been heavily influenced by how [spaCy tests their models](https://github.com/explosion/spacy-models/tree/master/tests#writing-tests).

## Running tests

As the tests are both: 
* Testing that the models can be created and installed via `pip` locally.
* Once created and installed the models function as expected. 

This has resorted into two test folders, as shown in [General Folder Structure](#general-folder-structure), `/model_function_tests` and `/model_creation_tests`. The `/model_creation_tests` tests the first bullet point and `/model_function_tests` tests the second bullet point.

### Model creation tests

As the `/model_function_tests` require the installed models that are created from `/model_creation_tests` the `/model_creation_tests` tests are ran first whereby the models created will be installed to a virtual environment that will be saved to `./temp_venv` **NOTE** `./temp_venv` is assumed to not exist, an error will occur if the directory does exist, unless you specify the `--overwrite` flag which will first delete the directory if it exists and then re-create.

``` bash
pytest --virtual-env-directory=./temp_venv ./model_creation_tests
```

Using the overwrite flag, which will first delete `./temp_venv` if it exists:

``` bash
pytest --virtual-env-directory=./temp_venv --overwrite ./model_creation_tests
```

This command can be ran as a `make` command:

``` bash
make model-creation-tests
```

### Model function tests

By separating these tests into two different test folders it allows the virtual environment to be cached, which allows the second set of tests, `/model_function_tests`, to be ran as many times as you like without having to re-create the virtual environment.

``` bash
source ./temp_venv/.env/bin/activate # Used to activate the virtual environment
pytest ./model_function_tests
deactivate
```

### All tests

There is a make command that will run all tests:

``` bash
make run-all-tests
```

### Caching models and virtual environment

This is a **two step process**:

1. To run the tests without creating the models and installing them each time we can cache both the models and the virtual environment that was used to install the models. Run the following replacing `MODELS_DIRECTORY` and `VIRTUAL_ENV_DIRECTORY` with a path to an empty directory:

``` bash
pytest --model-cache-directory=MODELS_DIRECTORY --virtual-env-directory=VIRTUAL_ENV_DIRECTORY
```

e.g.
``` bash
pytest --model-cache-directory=./models --virtual-env-directory=./temp_venv
```

2. Any future tests we can then run the following, which will not re-create or re-install the models and just use what we have cached:

``` bash
pytest --model-cache-directory=MODELS_DIRECTORY --virtual-env-directory=VIRTUAL_ENV_DIRECTORY --do-not-reinstall-packages
```

e.g.
``` bash
pytest --model-cache-directory=./models --virtual-env-directory=./temp_venv --do-not-reinstall-packages
```

**Note** if you want to test the model creation process you cannot use this caching process, but if you just want to test the models are working correctly or testing new tests for these models then this is great for those tests.



## Model deployment lifecycle

Each model is created using a spaCy [configuration](https://spacy.io/api/data-formats#config) and [meta](https://spacy.io/api/data-formats#meta) file, of which we can have more than one model for each language. These configuration and meta files are automatically created and stored for each model within their own model folder, which is named after the model using the model naming conventions specified in the main [README](./README.md#model-naming-conventions). These model folders are then stored within their relevant language folder, e.g. if it is Welsh model then it would be in the [./languages/cy/ folder.](./languages/cy/) Once the configuration and meta files are created the associated spaCy model is created and then uploaded to this repository as a [GitHub release](https://github.com/UCREL/pymusas-models/releases).

Below is a list of steps outlining the scripts that need to be ran to create these models with some extra detail on what each script does:

1. [scripts/create_config_and_meta_files.py](./scripts/create_config_and_meta_files.py). This script creates the configuration and meta files for each model using the model meta data we store on each language, within the [language_resources.json](./language_resources.json) file (for more information of the [language_resources.json](./language_resources.json) file see the [Language Resource Meta Data section](#language-resource-meta-data)).
2. 


## Language Resource Meta Data

Language resource meta data is stored in the [language_resources.json file](./language_resources.json), it is used by the [scripts/create_config_and_meta_files.py](./scripts/create_config_and_meta_files.py) script to create the models and their associated meta data file. The structure of the JSON file is the following:

``` JSON
{
    "Language one BCP 47 code": {
        "resources":[
            {
                "data type": "single", 
                "url": "PERMANENT URL TO RESOURCE"
            }, 
            {
                "data type": "mwe", 
                "url": "PERMANENT URL TO RESOURCE"
            }
        ],
        "model information": {
            "POS mapper": "POS TAGSET"
        },
        "language data": {
            "description": "LANANGUAGE NAME",
            "macrolanguage": "Macrolanguage code",
            "script": "ISO 15924 script code"
        }
    },
    "Language Two BCP 47 code" : {
        "resources":[
            {
                "data type": "single", 
                "url": "PERMANENT URL TO RESOURCE"
            }
        ],
        "model information": {
            "POS mapper": null
        },
        "language data":{
            "description": "LANANGUAGE NAME",
            "macrolanguage": "Macrolanguage code",
            "script": "ISO 15924 script code"
        }
        
    },
    ...
}
```

* The [BCP 47 code](https://www.w3.org/International/articles/language-tags/) of the language, the [BCP47 language subtag lookup tool](https://r12a.github.io/app-subtags/) is a great tool to use to find a BCP 47 code for a language.
  * `resources` - this is a list of resource files that are associated with the given language. There is no limit on the number of resources files associated with a language.
    * `data type` value can be 1 of 2 values:
      1. `single` - The `url` value has to be of the **single word lexicon** [file format](https://github.com/UCREL/Multilingual-USAS#single-word-lexicon-file-format).
      2. `mwe` - The `url` value has to be of the **Multi Word Expression lexicon** [file format](https://github.com/UCREL/Multilingual-USAS#multi-word-expression-mwe-lexicon-file-format).
    * `url` - permanent URL link to the associated resource.
  * `model information` - this is data that helps to create the model given the resources and the assumed NLP models, e.g. POS tagger, that will be used with the PyMUSAS model.
    * `POS mapper` - A mapper from that maps from the POS tagset of the tagged text to the POS tagset used in the lexicons. The mappers used are those from within the [PyMUSAS mappers module.](https://ucrel.github.io/pymusas/api/pos_mapper) We currently assume that each resource associated with the model uses the same POS tagset in the lexicon, this is a limitation of this model creation framework rather than the PyMUSAS package itself.
  * `language data` - this is data that is associated with the `BCP 47` language code. To some degree this is redundant as we can look this data up through the `BCP 47` code, however we thought it is better to have it in the meta data for easy lookup. All of this data can be easily found through looking up the `BCP 47` language code in the [BCP47 language subtag lookup tool](https://r12a.github.io/app-subtags/)
    * `description` - The `description` of the language code.
    * `macrolanguage` - The macrolanguage tag, **note** if this does not exist then give the [primary language tag](https://www.w3.org/International/articles/language-tags/#language), which could be the same as the whole `BCP 47` code. The `macrolanguage` tag could be useful in future for grouping languages.
    * `script` - The [ISO 15924 script code](https://www.w3.org/International/articles/language-tags/#script) of the language code. The `BCP 47` code by default does not always include the script of the language as the default script for that language is assumed, therefore this data is here to make the default more explicit.

Below is an extract of the [./language_resources.json](./language_resources.json), to give as an example of this JSON structure:

``` JSON
{
    "cmn": {
        "resources":[
            {
                "data type": "single", 
                "url": "https://raw.githubusercontent.com/UCREL/Multilingual-USAS/69477221c3feaf8ab2c2033abf430e5c4ae1d5ce/Chinese/semantic_lexicon_chi.tsv"
            }, 
            {
                "data type": "mwe", 
                "url": "https://raw.githubusercontent.com/UCREL/Multilingual-USAS/69477221c3feaf8ab2c2033abf430e5c4ae1d5ce/Chinese/mwe-chi.tsv"
            }
        ],
        "model information": {
            "POS mapper": "UPOS"
        },
        "language data": {
            "description": "Mandarin Chinese",
            "macrolanguage": "zh",
            "script": "Hani"
        }
    },
    "nl" : {
        "resources":[
            {
                "data type": "single", 
                "url": "https://raw.githubusercontent.com/UCREL/Multilingual-USAS/69477221c3feaf8ab2c2033abf430e5c4ae1d5ce/Dutch/semantic_lexicon_dut.tsv"
            }
        ],
        "model information": {
            "POS mapper": "UPOS"
        },
        "language data":{
            "description": "Dutch, Flemish",
            "macrolanguage": "nl",
            "script": "Latn"
        }
        
    },
    ...
}
```