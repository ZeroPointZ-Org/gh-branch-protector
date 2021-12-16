# gh-branch-protector

## Automatic branch protection of default branches

### Description:

When a new repository is created within your GitHub Organization - automatically set branch protections of the default branch. Initiated from a webhook event, processed by this simple web service, and notified with @ username mention in a new issue. The message details the key/value details of the protection parameters. (Default branches assumed to be named `main`)

![Issues](/assets/images/gh_issues_detail.png)

## Setup

The following prerequisite components are needed to support the webhook/webservice:
- [GitHub account](https://github.com)
- [GitHub Organization](https://docs.github.com/en/organizations)
- [Python3 v3.7.0+](https://www.python.org/downloads/)
- [Flask v1.1.1](https://flask.palletsprojects.com/en/1.1.x/installation/)
- [Ngrok v2.3.40](https://dashboard.ngrok.com/signup)

## Usage

### GitHub Account and Authorization

- Create a PAT (personal access token) for authentication. [Token](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/creating-a-personal-access-token)

- Select the repository and admin scopes when creating the Token

- Store the token in a system variable for later steps on your terminal
    `export GH_Token=<super_secret_token_password>`

- Change directory to the base of this repository folder

- modify app.py > user variable to your GitHub username

### Python / Flask / Ngrok

- Install Python based on the instructions related to your operating system
  - [Python3 v3.7.0+](https://www.python.org/downloads/)
  - Install Flask and required dependancies
    `pip install -r requirements.txt`

- Create a free account for [Ngrok](https://dashboard.ngrok.com/signup)
  - [Install Ngrok](https://ngrok.com/download)
  - log-in to the Ngrok dashboard in your browser

### Binding Flask and Ngrok

- Open 2 terminal sessions and navigate to the base of this repo

- Run the flask server in the first terminal session 
    - `flask run --host=0.0.0.0 &`
    - *Flask runs in Development Server mode by defualt, not intended for production*

- Run Ngrok in the second terminal session
    `./ngrok http 5000 &`
    - Copy the forwarding endpoint information for later steps
    - Verify the tunnel status and forwarding address in the ngrok dashboard

### GitHub Organization - Webhook

![Webhook](/assets/images/gh_webhook.png)

- Log-in to the GitHub Organization account
    - Navigate to `Settings` > `Webhooks`
    - Click the `add webhook` button
    - Update the `Payload` field with the http forwarding endpoint from ngrok
    - Select the option `application/json` in the content type drop-down
    - Select the `Let me select individual events` radio button
    - Select the `Repositries` events in the list
    - Select the `Active` event trigger button at the bottom of the list
    - Click the Save/Update button

### Create a new repository in the GitHub Organization account

![New repo](/assets/images/create_new_branch.png)

- Click the `New` button to create a new repository
    - Assign a name to the repository
    - Provide a brief description (optional)
    - Mark the repository as public
    - Select the radio button to `Add a README.md`
    - Click the `Create repository` button

### Validate the new branch protections on the default branch:

Once you click the `Create repository` button in the previous step, the webhook event will be triggered on the `create` event of the json payload. The json payload will `POST` to the publically exposed ngrok endpoint running in previous steps. The payload will be available to Flask by the binding (also in previous steps). The simple python webservice attached to flask will evaluate the json payload, create the branch protection attributes response to `PUT` to the GitHub API v3 `branches` endoint, and finally `POST` a new issue to the GitHub API v3 `issues` endpoint with the protection attributes along with a @ mention to the user.

- Terminal session 1 - running flask
  - `POST` payload transaction with a 200 status code

- Terminal session 2 - running ngrok
  - HTTP Requests transaction with matching `POST` and 200 responses

- Ngrok dashboard in browser window
  - Open a browser tab to Ngrok Inspect on `localhost:4040`
  - Verify the request and payload

- GitHub Organizations dashboard
  - Click into the newly created repo
  - Navigate to `Settings > Branches > Branch protection rules`
  - Verify the branch protection rule applies to the default branch
  - Navigate to `Issues`
  - Verify a new issue is created with protection rules information and user @ mention

## Attributions

- [Github Satellite 2020: Building GitHub Integrations with webhooks and REST](https://www.youtube.com/watch?v=wcxOJq9YemE&list=FL11qguAv2b2obKLbhn_1i_g)

- [GitHub APIv3 documentation](https://developer.github.com/v3/)

- [Code example](https://github.com/zkoppert/Auto-branch-protect)

- [Building Tools with GitHub by Chris Dawson (and Ben Straub) - O'Reilly books](https://www.oreilly.com/library/view/building-tools-with/9781491933497/)