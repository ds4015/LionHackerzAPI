# LionHackerz - Job Search API

Our project tasks/to-do lists and completion can be found on Trello at:

	https://trello.com/b/tavRkgxR/job-matching-api

 A deployed version (running app) can be found on the Google Cloud Platform at:
 	
  	http://34.69.114.32/

A demoable app/client can be found in the following repo:

	https://github.com/ds4015/w4156-app

 CI is set up to build and run tests, coverage, static analysis, and style check on push and pull requests. It is set up in Actions as a workflow named "CMake on a single platform". Click on a workflow run and scroll down to <b>Artifacts</b> section to download reports and outputs produced during runtime. 
 
 Some reports and outputs produced from a previous workflow run is in this repo in <b>workflow_output/</b>. Coverage, style checker, and static analysis report reports from a previous manual run is in <b>manual_output/</b>. Note we are using the `--decisions` flag in gcovr to get the actual code-level branches coverage rate, so we are aiming to achieve 85% <b>decisions</b> coverage rate in coverage_report_gcovr/coverage.html.

# Initial Setup

This API makes use of PostgREST. To install, follow step 2 here:

	https://docs.postgrest.org/en/v12/tutorials/tut0.html

The database queries make use of cURL, so if you don't have it installed on your
system, see here:

	https://curl.se/

For coverage report generation, you will need lcov, which can be installed with:

	brew install lcov

The matching algorithm uses WordNet 3.0 to build a list of synonyms for word
comparison between user profile and job listing.  To install this, download
WordNet 3.0 from:

	https://wordnet.princeton.edu/download/current-version

Place the uncompressed folder in the external_libraries directory of this API.

WordNet needs the tcl-tk package to be installed.  Install tcl-lk on Mac using:

    brew install tcl-tk

If using Windows, this can be installed using ActiveTcl.


Find the path where you installed tcl-tk, in particular the lib folder. For me,
this was located in:

    /opt/homebrew/Cellar/tcl-tk/8.6.15/lib

This directory should contain the files tclConfig.sh and tkConfig.sh among others.

Copy this path and go back to the WordNet-3.0 folder in the external_libraries folder.  Run the following command:

	./configure --with-tcl=/path__to_tcl-tk_library --with-tk=/path_to_tcl_tk_library

replacing the path with the actual path to your tcl-tk lib folder.

Now before you build, you will need to make one small change.  There is a file 'stubs.c' in the WordNet-3.0/src
directory which has some outdated commands.  This will need to be replaced with the 'stubs.c' file included in
this repo in the external_libraries directory, which contains the fixed version. Copy this into WordNet-3.0/src:

	sudo cp stubs.c WordNet-3.0/src/stubs.c

Finally, run:

	sudo make
	sudo make install

Optional: 

To get rid of WordNet warnings when compiling, copy over the revised wn.h file
from the external_libraries root directory into external_libraries/WordNet-3.0/include
From the project root directory, run the following:

`sudo cp external_libraries/wn.h external_libraries/WordNet-3.0/include/wn.h`

This should complete the WordNet library installation. 

NEW (11/25/2024):

This API makes use of cpp_redis for task queueing instances of the job matcher.  To install, follow the instructions
found below.  Even though cpp_redis contains tacopie, it may be better to install tacopie first on its own before cpp_redis
to avoid errors:

	https://github.com/Cylix/tacopie/wiki/Installation

After that, you can install cpp_redis:

	https://github.com/cpp-redis/cpp_redis/wiki/Installation

If you run into errors about EXPORT(tacopie...), you can ignore if you've instlled tacopie first
and just run make followed by sudo make install.  It should still install this way. 

	git clone https://github.com/cylix/tacopie.git
 	cd tacopie
	mkdir build && cd build
 	cmake ..
  	make
   	sudo make install

If you run into an error about ::client::connect, simply remove the "::client" part of the main redis
connection function in the API sourece files main.cpp and test/testMain.cpp and it should work.

cpp_redis makes use of the nlohmann json package.  It can be installed on macOS with:

	brew intall nlohmann-json

For other platforms, see:

	https://github.com/nlohmann/json

When building the service in /build, if you get error such as "terminating due to uncaught 
exception of type cpp_redis::redis_error: connect() failure", then make sure redis is installed
and running:

`brew install redis`

`brew services start redis`

Optional: 

To get rid of warning messages from redis when compiling, copy the 'redis_logger.hpp' and 
'tacopie_logger.hpp' files from the external_libraries directory into your local install 
directories (on macOs, these should be /usr/local/include/cpp_redis/misc and 
/usr/local/include/tacopie/utils), removing the  prefixes reddis_ and tacopie_ 
(i.e., the copied versions should just be named logger.hpp in their respective 
destination folders). To do this, from external_directories in the project, run:

`sudo cp redis_logger.hpp /usr/local/include/cpp_redis/misc/logger.hpp`

`sudo cp tacopie_logger.hpp /usr/local/include/tacopie/utils/logger.hpp`

If your local libraries are installed somewhere else, use that path instead to copy over into.


NEW:

This app uses OpenSSL for API key generation. To install, download version 3.3.2 from https://openssl-library.org/source/, then go to openssl-3.3.2/, run `./Configure` and then `make`.

# Building and runing a local instance of the API Server

Follow the README.txt in /external_libraries to get Crow and Boost.

Install dependency: asio (`brew install asio` on mac).

Set up environment variables for Supabase DB named SUPABASE_URL and SUPABASE_API_KEY:

for eg. `vi ~/.bashrc`, add `export SUPABASE_URL=“{URL}”`, and add `export SUPABASE_API_KEY=“{API_KEY}”`, save and run `source ~/.bashrc`. If using VS Code or some other IDE terminal to run project, may need set them again inside the IDE terminal to make sure they are accessible.

To build project: go to /build and run: `cmake ..` and then `make`, which will generate an executable named LionHackerzProject.

To run app: go to /build and run: ./LionHackerzProject.

# AI Features

This API integrates with OpenAI's ChatGPT for optional AI-generated job listings.  In order to use this feature, an API key from OpenAI is necessary.  For more on this, see:

	https://platform.openai.com/docs/quickstart

With an API key set as an environmental variable and an active balance with OpenAI, functions which make use of AI will be available to perform queries and return AI-generated results.

# API Entry Points

## Updated work in progress API reference doc is [here](https://docs.google.com/document/d/1NSQ_4oRS3Y2LhI0e3Yp9fjCGNDtPFUAQwig-Lj07cNU/edit)

This service implements RESTful APIs. Requests either contain parameters or a JSON body. Responses are JSON objects.

### Base URL

http://0.0.0.0:18080

### Error response

If the HTTP response code is an error, the following response will be returned:

```json
{  
  "error": {  
    "code": ...,  
    "message": "..."  
  }  
}
```

### Authentication:

This service uses API keys to authenticate requests. Anthentication for API keys is done via HTTP Basic Auth. Use your API key for the basic auth username field and no password is needed. ...

<b>For team members</b>: Every team member is an admin. Admins can generate API keys to use for all admin privileges. To generate an API key as an admin, check your username and password in Supabase in the Admin table, then run the app and send a POST request to /signup, with parameter "role" as "admin", and use the username and password from the Admin table for the corresponding basic auth fields. Then your API key will be returned in the JSON response and will be stored in the database. Request authentication will be gradually implemented to check for API keys, first for admins, then for all kinds of permissions (eg. different client permissions).

### Endpoints:

GET /

GET /dbtest

	Tests of the custom database class

GET /getMatches

	Get a listing of job matches (for users)

POST /listing/create

	Create a new job listing (for employers)

POST /listing/generateAI

	Generate a set number of full job listings using AI

POST /listing/generateJobDescription

	Generate a job description using AI

PATCH /listing/changeField

	Change the job field of a listing

PATCH /listing/changePosition

	Change the job position field in a listing

PATCH /listing/changeJobDescription

	Change the job description field in a listing

PATCH /listing/changeFlex

	Change the boolean flexibility field in a listing

PATCH /listing/changeModernWorkspace

	Change the boolean workspace field in a listing

POST /makeUser

	Register a new user account

GET /getProfile

	Returns the profile of a given user to display alongside match results

Postman tests for these routes can be found on our Trello board located here:

    https://trello.com/c/2X1naGF8/29-postman-tests-export

### User Creation

The user is created using a JSON request to the /makeUser endpoint. The request body will need to have a name and email. The request needs to have a "dimensions" parameter, which contains the information about the user's desired pay, field, gender, etc. The user may or may not indicate their skills and interests, and they may or may not include information about their "augments" which specifies how important certain things like location, pay, etc for a job is to a user and this will be used in the matching algorithm.
```
Sample Request:
{
  "name": "SampleUser",
  "email": "sampleuser@email.com",
  "dimensions": {
    "loc": "New York",
    "field": "Software Engineering",
    "pay": 120000,
    "gender": true,
    "diversity": true,
    "mbti": "INTJ",
    "flexibility": true,
    "remote": true,
    "workspace": false
  },
  "skills": [
    { "name": "python" }
  ],
  "interests": [
    { "name": "drawing", "rank": 3 },
    { "name": "software development", "rank": 5 }
  ],
  "augments": [
    { "dim_id": "1", "importance": "very" },
    { "dim_id": "2", "importance": "somewhat" }
  ]
}
```

# Database Queries - For Team Members in Designing your Classes

Queries are performed using cURL formatted URLs to the database. You don't need
to worry about this for performing queries, as the Database class has an
overloaded query function to automatically format the URLs and perform the
queries and return the results. However, if you are interested in how the
queries are formatted with PostgREST and curl, see:

    https://postgrest.org/en/v10/api.html

See the commentary in Database.h for the query/insert functions for the parameters you
will need to pass in for your requests. When creating your classes, make sure
you include an instance of the Database class. This allows you to perform
queries/inserts inside your member functions as needed. You can store the results in
private instance variables to use as necessary.

# Current Database Tables:

    User:		id (auto created), created_at uname, email
    /* list of users/job seekers */
    
    Dimension:	dim_id (auto created), name, def_weight
    /* matching parameters with weights */

    Interest:       name, category
    
    Skill:          name, category

    Has_Interest:	id, name  (id from User, name from Interest)
    /* relation between User and Interest */

    Has_Skill:	id, name  (id from User, name from Skill)
    /* relation between User and Skill

    Has_Augment:	id, dim_id, weight_mod  (id from User, dim_id from Dimension)
    /* relation between User and Dimension */

    Listing:  lid (auto generated), created_at (auto generated),
                field, position, job_description, skill1_req, skill2_req,
                skill3_req, skill4_req, skill5_req, pay, job_flexibility,
                diverse_workforce, remote_available, personality_types,
                modern_building, mixed_gender, location
    /* job listing with matching parameters */

    Employer: eid (auto generated), company_name, size
    /* list of companies/employers */

    Creates: eid, lid  (eid from Employer, lid from Listing)
    /* relation between Employer and Listing */
                
