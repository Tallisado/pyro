pyrobot
=======

"PYRO" is a suite of several tools, which provide integrations between Teamcity, Sauce, Robot Framework and Selenium. 

## [Bot](https://github.com/Tallisado/pyrobot "PyroBot")
Pyrobot is a python script which wraps the robot framework toolchain provided by 'pybot'. It is the executionary step that pulls in PyroFramework and starts the tests that pull in the PyroLibrary. This component serves as the entry point to the suite, which houses the testcode and the archived results of the suite.

## [Library](https://github.com/Tallisado/pyrolibrary "PyroLibrary")
The PyroLibrary is a web testing library for Robot Framework that provides Sauce Integration, Browser Capability testing and Custom xtendable Selenium2Library methods. 

## [Factory](https://github.com/Tallisado/pyrofactory "PyroFactory")
PyroFactory is a python wrapper that provides integration between TeamCity's Sauce CI Plugin and PyroBot. The tool intersects the Sauce CI environment variables for Single and Multiple Browser tests, and creates a series of payloads to run against these vectors . The factory can also run standalone outside of teamcity (cmdline), where it can instantiate a local browser, or even direct the payload test code to a remote Sauce browser of your choosing. Once tests are executed, the results are concatinated by rebot to provide aggrigated results in a single log and report.

## Feature Set Highlights

#### Browser Capability
- Intersects Teamcity/Cmdline information to determine which browsers to run on; such as, local or remote (Sauce)
- SAUCE_ONDEMAND_BROWSERS (Multiple Sauce Browsers), SELENIUM_DRIVER (Single Sauce Browser), and BROWSER (CmdLine execution of Local and Solo Sauce Browser) environment variables are intersected
- Can be executed through Teamcity or Standalone on the commandline

#### Pybot Eenhanced Feature Set
- Spawns pybot processes in serial
- Stages each pybot test suite with the target browser
- Browser information is embedded into log

#### Rebot file management
- Concat  pybot's resulting output and reports

#### Sauce Information
- __TODO__ Teamcity tab captures the resulting testcode data, which provides a single glance update of the suite execution

#### Github and PIP Installation
- Both library and factory are installed through pip


## How it works:
1. A Teamcity build is started, and as such invokes; [Sauce CI - Plugin](http://saucelabs.com/teamcity/1 "SauceLabs Teamcity plugin") and Pyrobot, the build feature and build step respectively.
  * __Example Build__ Feature ![Build Feature](docs/img/teamcitybuildfeature_sauce.JPG?raw=true)
  * __Example Pyrobot__ ![Pyrobot](docs/img/teamcitybuild_sauce.JPG?raw=true)
2. Teamcity will start OnConnect (if selected)
3. Pyrobot executes the payload, either a single file or entire directory
  * If a directory is provided, the directory contents can be enumerated to allow the files to be ordered.
  * __Example Enumeration ![Pyrobot](docs/img/robotframework_enum_files.JPG?raw=true)
4. The results are placed into workspace under the pyrobot project eg:__pyrobot/workspace__ under a unique folder, after which the files are consolidated into a single report if multiple browsers were selected in build features
  * The unique folder under workspace is randomly created unless the WORKSPACE_UID is specified on the command line (allows TeamCity to Atrifact reference)
5. __TODO__ Sauce REST API integration to show results under 'Sauce Results' tab in teamcity build

## Feature Explanation:
### TeamCity Sauce Plugin interpretation
  * The SauceCI plugin provides environment variables, which execute multiple or single browsers based on the Suace CI Plugin selection within the build feature.
  * These environment vaiables are intersected and intrepreted to provide a remote selenium webdriver instantiation to robot framework
  * ```WORKSPACE_UID=WEEKLY_DEV_SM_%env.BUILD_NUMBER% BASE_URL=http://10.10.9.63 PAYLOAD=/mnt/wt/pyrobot_v1.1/pyrobot/dev/spec/02__pyrobot_library.txt python /mnt/wt/pyrobot_v1.1/pyrobot/pyrobot.py ```
  
### Execute standalone interpretation
  * Pyrobot can be executed from the commandline, using ant, or directly with the python script itself. When in standalone mode, the browser object is created based on the default values in __./PyrobotConfig__
  * Display Buffer: ``` vncserver :60 -geometry 1280x1024 ``` or ``` Xvfb ... ```
  * ```BROWSER='chrome' BASE_URL=http://10.10.9.63 PAYLOAD=/mnt/wt/pyrobot_v1.1/pyrobot/dev/spec/02__pyrobot_library.txt python /mnt/wt/pyrobot_v1.1/pyrobot/pyrobot.py ```
  * ```BROWSER='Linux,chrome,32' WORKSPACE_UID=WEEKLY_DEV_SO_%env.BUILD_NUMBER% BASE_URL=http://10.10.9.63 PAYLOAD=/mnt/wt/pyrobot_v1.1/pyrobot/dev/spec/02__pyrobot_library.txt python /mnt/wt/pyrobot_v1.1/pyrobot/pyrobot.py ```


### Pybot wrapper
  * Pybot is executed for each payload specified, and all selected browsers in the Build Feature in Sauce CI.
  * The Selenium2Library is augmented to provided integration to the Sauce service (see Pyrobot Robot Framework)

#### Pybot Configuration Examples
  * ``` SAUCE_USERNAME = "yourname" ```
  * ``` SAUCE_ACCESSKEY = "yourkey" ```
  * ``` DEFAULT_SAUCEURL = "sauce-ondemand:?username=%s&access-key=%s&os=Windows 2012 R2&browser=%s&browser-version=11&max-duration=null&idle-timeout=null" ```
    1. when executed in standalone, this remote webdriver is instanciated if using sauce
  * ``` DEFAULT_SOLO_BROWSER = 'firefox' ```
    1. when executed in standalone, this remote webdriver is instanciated if using local browser
  * ``` DEFAULT_BROWSER_DISPLAY = ":60" ```
  * ``` BASE_URL = "http://www.google.ca" ```
    1. when not provided through ant/python, base_url default here
  * ``` WORKSPACE_HOME = "/mnt/wt/pyrobot_v1.1/pyrobot/workspace/" ```
  
#### Pyrobot Robot Framework
1. Reference the resource file: 
  * ``` Resource       ../resources/resource.txt ```
2. Various Invocations:
  1. 	``` Open Pyrobot	  http://www.google.ca                              __Local Default Browser TestLevel URL__ ```
  2. 	``` Open Pyrobot 	  http://www.google.ca      chrome                  __Local Non-Default Browser TestLevel URL__ ```
  3. 	``` Open Pyrobot 	  ${PYRO_BROWSER}           http://www.google.ca    __Sauce Browser TestLevel URL__ ```
  4. 	``` Open Pyrobot	  ${PYRO_BROWSER}                                   __Sauce Browser BaseUrl URL OnConnect__ ```
    * above this base_url is OnConnect only

## Robot Framework - Custom Selenium API
1. TODO