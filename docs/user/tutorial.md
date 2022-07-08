# Tutorial

## Situation
You have done some research and want to make the code you created available to other scientists, so they can take a better look at your results. 
For this, ViPLab was integrated into your institutions management system for research data (see Chapter on [Integration](../admin/integration.md))

---

## Steps
1. Create a Docker Container of your Research Software, that can be used by ViPLab
2. Write a Computation Template
3. See the Result in the ViPLab Frontend

---

### Easy Example
For this Tutorial we have created an [easy example code](https://github.com/VirtualProgrammingLab/viplab-example-docker), that just outputs information that was passed to the script using an INI-file and a C-file. 
For the last seven lines, some example output is copied inside the docker container, such that they are sent back as part of the result. 

The python script looks like this: 
``` python title="run.py"
import configparser
import shutil
import os

cwd = os.getcwd()

config = configparser.ConfigParser(allow_no_value=True)
config_file = cwd + '/data/shared/params.ini'
config.read(config_file)

print("Your name is \033[0;35m", config['about you']['name'], "\033[0m")
print("Your current age is", config['about you']['age'])
print("Your Christmas Wish is:", config['about you']['christmasWish'])
print("How hot do you like to drink your Coffee?", config['coffee preference']['coffeeTemperature'])
print("You like: \033[0;32m", config['about you']['likedThings'], "\033[0m")
print("Your favorite Programming Language is", config['about you']['favoritePL'])
print("Your look in the frige:", config['about you']['fridge'])
print("You would dance in the kitchen to:", config['about you']['dancing'])
print("You dislike: \033[0;31m", config['about you']['dislikedThings'], "\033[0m")
print("Your three random numbers are:", config['about you']['randomNumbers'])

code_file = cwd + '/data/shared/code.c'
f = open(code_file, "r")
codeSnippet = f.read()
print("The Code you entered:")
print(codeSnippet)
f.close()

shutil.copyfile(cwd + "/data/output/earth.vtp", cwd + "/data/shared/earth.vtp")
shutil.copyfile(cwd + "/data/output/coffee-temp.csv", cwd + "/data/shared/coffee-temp.csv")
shutil.copyfile(cwd + "/data/output/coffee.jpg", cwd + "/data/shared/image-coffee.jpg")
shutil.copyfile(cwd + "/data/output/dance.png", cwd + "/data/shared/image-dance.png")
shutil.copyfile(cwd + "/data/output/uris.txt", cwd + "/data/shared/uris.txt")
shutil.copyfile(cwd + "/data/shared/params.ini", cwd + "/data/shared/input.ini")
shutil.copyfile(cwd + "/data/shared/code.c", cwd + "/data/shared/input.c")
```

An example for the input-file called params.ini could look like this: 
``` ini title="params.ini"
[coffee preference]
coffeeTemperature=75

[about you]
likedThings=programming,music,books
favoritePL=Java
fridge=never
dancing=Last Christmas
dislikedThings=Spiders
randomNumbers=25,50,75
name=Kathryn
christmasWish=COFFEE! In that nebula!
age=36
```

Additionally the the input-file called code.c could look like this: 
``` c title="code.c"
int main(int argc, char **argv) { 
// Print 'Hello World' 
}
```

!!! tip inline end
    Remember to move to the correct directory using `cd [path-to-directory]/viplab-example-docker`, meaning the freshly cloned [example-repository](https://github.com/VirtualProgrammingLab/viplab-example-docker)!

To try out how the script works, run `run.py`:

Calling...
``` sh
python run.py
```

... will result in the following output on the console: 

``` console
Your name is Kathryn
Your current age is 36
Your Christmas Wish is: COFFEE! In that nebula!
How hot do you like to drink your Coffee? 75
You like:  programming,music,books
Your favorite Programming Language is Java
Your look in the frige: never
You would dance in the kitchen to: Last Christmas
You dislike:  Spiders
Your three random numbers are: 25,50,75
The Code you entered:
int main(int argc, char **argv) {
// Print 'Hello World'
}
```

### 1. Create a Docker Container of your Research Software, that can be used by ViPLab

If you are not familliar with Docker, follow this link to the official website to find the right Download for your system and take a look at the docs: [Docker Website](https://www.docker.com/).
There is also a [first introduction to Docker](https://docs.docker.com/get-started/) available, with a few explainations and an image to try out. 
To create and runs your first Docker sample application, you can follow this [Tutorial](https://docs.docker.com/get-started/02_our_app/). 

After you set up Docker, you can start writing a Dockerfile for your research software. 
The file is placed in the same folder as your code. 

For the example code above, the Dockerfile would look like this: 

``` docker title="Dockerfile"
FROM python:latest
COPY *.py /data/bin/
COPY earth.vtp /data/output/earth.vtp
COPY coffee-temp.csv /data/output/coffee-temp.csv
COPY coffee.jpg /data/output/coffee.jpg
COPY dance.png /data/output/dance.png
COPY uris.txt /data/output/uris.txt
CMD ["python", "/data/bin/run.py"]
```
!!! info "Explaination of the Dockerfile"
    First the latest Ubuntu-Docker-Image is selected as a starting point. After this, our files are copied to specific folders inside the container. Then `run.py` is being executed.

After creating the file, open a terminal and go to the directory where your files are located, if you haven't done so, yet. 
To build the container image, run the following command:

``` docker
docker build -t viplab-example-image .
```

After the build is finished, you can now start the container using...

!!! tip inline end
    If you are on Windows, please use Powershell to execute the run command

```
docker run -v ${PWD}/shared:/data/shared -it viplab-example-image
```

... after which you will see the output shown in section [Easy Example](#easy-example). 

### 2. Write a Computation Template

To write a Computation Template you have to make several considerations. 
For the Template, you need all the input for your application: Commandline arguments and input-files. 
Using the input-files, you can also let the users make changes to code. 
Files can consist of multiple parts. 
In a part, you can also show code to the user, that can't be modified, to give the user more information, by setting the "access" to "visible". 
Code can be modified by setting "access" to "modifiable", or hidden by "invisible". 
If you want the user to input their changes by using form-elements, for example such that non-programmers can also try out your application, this can be done by setting "access" to "template". 
Then you can choose form-elements for value-selection, just as you can do it for commandline arguments. 
As an example: 
For the selection of numbers, you could use an input-field of type number or a slider. 
To select multiple text-values, you can use either checkbox or a dropdown and to give the user more freedom, you can also choose an input-field with type text or an editor for generating the necessary input.

To input the user changes into your program, the "content" of the parts can be set as a [Handlebars.js](https://handlebarsjs.com/guide/) template. 
For the easy example, this feature is used to generate both input files: `params.ini` and `code.c`. 
In this case, `params.ini` was split into two parts that is later merged to one file internally by ViPLab. 

Using a handlebars expression, one can replace parts of the template with user input, when it is executed. 
In this case `{{__sliderSingle__}}` is replaced by the input from the user. 

``` ini title="params.ini Part 1 as Handlebars.js-Template"
[coffee preference]
coffeeTemperature={{__sliderSingle__}}
```

The second part of the `params.ini`-Template is more complicated, as the input contains lists. 
For more detailed information on each handlebars functionality visit the link above. 
For example using `each`, one can iterate over an array, `if` enables one to make conditionals and `@last` helps you to refer to the last item of an array. 

To illustrate this a bit better, let's take a look at the line `likedThings={{#each __checkbox__}}{{#if @last}}{{.}}{{else}}{{.}}, {{/if}}{{/each}}` inside the template. 
The `likedThings` can have multiple values and as seen above, it has to be a list of items separated by a comma: `likedThings=programming,music,books`. 
In this example, we decided to supply a few values to the users using a checkbox, such that they can select from a limited number of items. 
This also prevents the user from entering incorrect or dangerous (as in malicious) input.
As explained in [Computation Task](../viplab-3.0/computation_task.md), the output of a checkbox-element consists of an array of strings. 
This array needs to be converted to a comma-separated list. 

``` ini title="params.ini Part 2 as Handlebars.js-Template"
[about you]
likedThings={{#each __checkbox__}}{{#if @last}}{{.}}{{else}}{{.}}, {{/if}}{{/each}}
favoritePL={{__radioButton__}}
fridge={{__dropdownSingle__}}
dancing={{#each __dropdownMultiple__}}{{#if @last}}{{.}}{{else}}{{.}}, {{/if}}{{/each}}
dislikedThings={{#each __toggle__}}{{#if @last}}{{.}}{{else}}{{.}}, {{/if}}{{/each}}
randomNumbers={{#each __sliderMultiple__}}{{#if @last}}{{.}}{{else}}{{.}}, {{/if}}{{/each}}
name={{{__inputTextWOMaxlength__}}}
christmasWish={{__inputTextWMaxlength__}}
age={{__inputNumber__}}
```

``` ini title="code.c as Handlebars.js-Template"
{{{__default__}}}
```


Here, you can see the Copmutation Template asking for the input of the user: 

``` json title="Computation Template of the Example"
{ 
  "identifier"  : "11483f23-95bf-424a-98a5-ee5868c85c3f", 
  "version" : "3.0.0",
  "metadata": { 
    "displayName" : "Parameters Example",  
    "description" : "This is a 'Hello World' example showing the usage of parameters. Please introduce yourself so that the Hello World-Container can print your information...",
    "output" : {
      "viewer" : ["CSV", "ParaView", "Image"]
    }
  },
  "environment" : "Container", 
  "files" : 
  [
    { 
      "identifier": "22483f42-95bf-984a-98a5-ee9485c85c3f", 
      "path"      : "params.ini",                              
      "metadata"  : 
        {  
          "syntaxHighlighting": "ini"                   
        },
      "parts" : 
      [
        {
          "identifier": "f3fc4404-3529-4962-b252-47bc4ddd02a1",
          "access": "template",
          "metadata": {
            "name": "Parameter in part",
            "emphasis": "low"
          },
          "parameters" : 
          [
            {
              "mode" : "any",
              "identifier" : "__sliderSingle__", 
              "metadata" : {
                "guiType" : "slider",
                "name": "Temperature",
                "vertical": false,
                "description" : "How hot do you like your coffee? (in degrees Celsius) - Tip: Typical Serving Temperature lies between 65 and 70 Degrees"
              },
              "default": [
                75
              ],
              "min": 0,
              "max": 90,
              "step": 5,
              "validation": "range"
            }
          ],
          "content": "W2NvZmZlZSBwcmVmZXJlbmNlXQpjb2ZmZWVUZW1wZXJhdHVyZT17e19fc2xpZGVyU2luZ2xlX199fQ"
        },
        {
          "identifier": "ceb051d8-b50c-4814-983a-b9d703cae0c6",
          "access"    : "template",
          "metadata"  :
              { 
                "name"      : "params.ini file part"
              },
          "parameters":
          [
            {
              "mode" : "fixed",
              "identifier" : "__checkbox__", 
              "metadata" : {
                "guiType": "checkbox",
                "name": "Things I like",
                "description" : "Select things you like"
              },
              "options": [
                {
                  "text" : "Programming",
                  "value" : "programming",
                  "selected" : true
                },
                {
                  "value" : "music"
                },
                {
                  "value" : "books"
                }
              ],
              "validation": "minone"
            }, 
            {
              "mode" : "fixed",
              "identifier" : "__radioButton__", 
              "metadata" : {
                "guiType": "radio",
                "name": "Favorite PL",
                "description" : "Select your favorite programming language"
              },
              "options": [
                {
                  "value" : "C"
                },
                {
                  "value" : "Java",
                  "selected" : true
                },
                {
                  "value" : "Haskell",
                  "disabled" : true
                },
                {
                  "text" : "Sssss... Python ...ssssS",
                  "value" : "Python"
                }
              ],
              "validation": "oneof"
            },
            {
              "mode" : "fixed",
              "identifier" : "__dropdownSingle__", 
              "metadata" : {
                "guiType": "dropdown",
                "name": "Fridge",
                "description" : "How often do look into the fridge a day?"
              },
              "options": [
                {
                  "value" : "Please choose one",
                  "disabled" : true
                },
                {
                  "value" : "never",
                  "selected" : true
                },
                {
                  "text" : "1 a day",
                  "value" : "Once a day"
                },
                {
                  "value" : "Twice a day"
                },
                {
                  "value" : "Three times a day"
                },
                {
                  "value" : "More than three times a day"
                }
              ],
              "validation": "oneof"
            }, 
            {
              "mode" : "fixed",
              "identifier" : "__dropdownMultiple__", 
              "metadata" : {
                "guiType": "dropdown",
                "name": "Dance Time",
                "description" : "To which songs would you dance in the kitchen?"
              },
              "multiple": true,
              "options": [
                {
                  "value" : "Please choose multiple",
                  "disabled" : true
                },
                {
                  "text" : "Last Christmas (aka the one that drives everybody else crazy)",
                  "value" : "Last Christmas",
                  "selected" : true
                },
                {
                  "value" : "White Christmas"
                },
                {
                  "value" : "Winter Woderland"
                },
                {
                  "value" : "Thats Christmas To Me", 
                  "selected" : true
                },
                {
                  "value" : "O Come All Ye Faithful", 
                  "disabled" : true
                }
              ], 
              "validation": "anyof"
            }, 
            {
              "mode" : "fixed",
              "identifier" : "__toggle__", 
              "metadata" : {
                "guiType": "toggle",
                "name": "NO!",
                "description" : "What do you dislike?"
              },
              "options": [
                {
                  "value" : "Spiders",
                  "selected" : true
                },
                {
                  "text" : "All kinds of Bugs (also the ones living in your Computer)",
                  "value" : "All kinds of Bugs"
                },
                {
                  "value" : "I never dislike anything!"
                }
              ], 
              "validation": "anyof"
            }, 
            {
              "mode" : "any",
              "identifier" : "__sliderMultiple__", 
              "metadata" : {
                "guiType" : "slider",
                "name": "random numbers",
                "vertical": true,
                "description" : "Choose three random numbers to be output by the container"
              },
              "default": [
                25,
                50,
                75
              ],
              "min": 0,
              "max": 100,
              "step": 5,
              "validation": "range"
            },
            {
              "mode" : "any",
              "identifier" : "__inputTextWOMaxlength__", 
              "metadata" : {
                "guiType" : "input_field",
                "type": "text",
                "name": "name",
                "description" : "Enter your name"
              },
              "default" : [""],
              "validation": "none"
            },
            {
              "mode" : "any",
              "identifier" : "__inputTextWMaxlength__", 
              "metadata" : {
                "guiType" : "input_field",
                "type": "text",
                "name": "Christmas Wish",
                "description" : "Enter what you wish for at christmas"
              },
              "maxlength": 200,
              "default" : [""],
              "validation": "pattern", 
              "pattern": "^[A-z_! ]+$"
            },
            {
              "mode" : "any",
              "identifier" : "__inputNumber__", 
              "metadata" : {
                "guiType" : "input_field",
                "type": "number",
                "name": "Age",
                "description" : "Enter your current age"
              },
              "default": [25],
              "min": 0,
              "max": 100,
              "step": 1,
              "validation": "range"
            }
          ],
          "content"   : "W2Fib3V0IHlvdV0NCmxpa2VkVGhpbmdzPXt7I2VhY2ggX19jaGVja2JveF9ffX17eyNpZiBAbGFzdH19e3sufX17e2Vsc2V9fXt7Ln19LCB7ey9pZn19e3svZWFjaH19DQpmYXZvcml0ZVBMPXt7X19yYWRpb0J1dHRvbl9ffX0NCmZyaWRnZT17e19fZHJvcGRvd25TaW5nbGVfX319DQpkYW5jaW5nPXt7I2VhY2ggX19kcm9wZG93bk11bHRpcGxlX199fXt7I2lmIEBsYXN0fX17ey59fXt7ZWxzZX19e3sufX0sIHt7L2lmfX17ey9lYWNofX0NCmRpc2xpa2VkVGhpbmdzPXt7I2VhY2ggX190b2dnbGVfX319e3sjaWYgQGxhc3R9fXt7Ln19e3tlbHNlfX17ey59fSwge3svaWZ9fXt7L2VhY2h9fQ0KcmFuZG9tTnVtYmVycz17eyNlYWNoIF9fc2xpZGVyTXVsdGlwbGVfX319e3sjaWYgQGxhc3R9fXt7Ln19e3tlbHNlfX17ey59fSwge3svaWZ9fXt7L2VhY2h9fQ0KbmFtZT17e3tfX2lucHV0VGV4dFdPTWF4bGVuZ3RoX199fX0NCmNocmlzdG1hc1dpc2g9e3tfX2lucHV0VGV4dFdNYXhsZW5ndGhfX319DQphZ2U9e3tfX2lucHV0TnVtYmVyX199fQ"
        }
      ]
    },
    { 
      "identifier": "22483f42-95bf-984a-98a5-ee9485c85c31", 
      "path"      : "code.c",                              
      "metadata"  : 
        {  
          "syntaxHighlighting": "c_cpp"                   
        },
      "parts" : 
      [
        {
          "identifier": "f3fc4404-3529-4962-b252-47bc4ddd02a2",
          "access": "template",
          "metadata": {
            "name": "Parameter in part",
            "emphasis": "low"
          },
          "parameters" : 
          [
            {
              "mode" : "any",
              "identifier" : "__default__", 
              "metadata" : {
                "guiType" : "editor", 
                "name": "code",
                "description" : "Enter some code"
              },
              "default": ["aW50IG1haW4oaW50IGFyZ2MsIGNoYXIgKiphcmd2KSB7IA0KLy8gUHJpbnQgJ0hlbGxvIFdvcmxkJyANCn0"],
              "validation": "none"
            }
          ],
          "content": "e3t7X19kZWZhdWx0X199fX0"
        }
      ]
    }
  ],
  "configuration" :
    { "resources.image"  : "name://viplab-example-image",
      "resources.volume" : "/data/shared",
      "resources.memory" : "1g",
      "resources.numCPUs" : 1
    }
}
```

!!! error "TODO"
    GUI CT Creator

For more detail on the structure of the Computation Template, take a look at the [Developer Guide](../viplab-3.0/computation_template.md).

### 3. See the Result in the ViPLab Frontend

Using the above created Computation Template, the Frontend will look like this: 

<figure markdown>
  ![ViPLab Frontend showing the Example](../images/frontend-example-mustache.png)
  <figcaption>ViPLab Frontend showing the Example</figcaption>
</figure>

After clicking the Play-Button, the Container will be executed by the Frontend and the Result will be returned, as seen in the Images below:

!!! success "Result after Running the Example using ViPLab"

    === "Result - Stdout"

        <figure markdown>
          ![ViPLab Frontend showing the Result](../images/example-result-1.png)
          <figcaption>ViPLab Frontend showing the Stdout</figcaption>
        </figure>

    === "Result - Stderr"

        <figure markdown>
          ![ViPLab Frontend showing the Result](../images/example-result-2.png)
          <figcaption>ViPLab Frontend showing the Stderr</figcaption>
        </figure>

    === "Result - Files"

        <figure markdown>
          ![ViPLab Frontend showing the Result](../images/example-result-3.png)
          <figcaption>ViPLab Frontend showing the Files</figcaption>
        </figure>

    === "Result - Download Links"

        <figure markdown>
          ![ViPLab Frontend showing the Result](../images/example-result-4.png)
          <figcaption>ViPLab Frontend showing the Download Links</figcaption>
        </figure>