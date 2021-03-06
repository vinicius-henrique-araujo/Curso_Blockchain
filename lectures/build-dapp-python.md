# Building a DApp in Python
In this lecture we will build a DApp frontend in the Python language, using the [Flask](https://flask.pocoo.org) framework.

## Getting started
For this project we will need to install a few python packages.
```bash
$ pip3 install flask, web3
$ pip3 install "eth-utils>=1.2.0"
```
Besides Flask, we also installed [web3.py](https://web3py.readthedocs.io/en/stable/index.html), which takes care of the comunication of the frontend with the Ethereum blockchain (via a node).


## Developping the Flask App
First we create a directory for our App. Let's call it `py-frontend`:
```bash
$ mkdir py-frontend
$ cd py-frontend
```
Inside this directory we will create a module named `DApp.py`:
```python
from flask import Flask
from flask import render_template
import web3
from web3 import Web3, HTTPProvider, TestRPCProvider
from web3.contract import ConciseContract
import json


w3 = Web3()  # Auto-detects connection settings
contract_address = Web3.toChecksumAddress('0xc6c817f52322a3edf269883eb1d612cb3fa096a2')
with open('../FunnyToken/build/contracts/FunnyToken.json') as f:
    token_artifact = json.load(f)

token_contract_instance = w3.eth.contract(address=contract_address, abi=token_artifact['abi'])
account = w3.eth.accounts[0]

app = Flask(__name__)

@app.route('/')
def hello_world():
    bal = token_contract_instance.functions.balanceOf(account).call()
    return render_template('index.html', balance=bal, address=contract_address, account=account)
```
This is our first `minimal` version of the DApp. But we are still going to improve it. You can find the most up-to-date version on the [ICO-playground](https://github.com/fccoelho/ICO-playground/tree/2018_project) repository. 

For this to Work, we will also need the `index.html` template: If you are not familiar with web development, an HTML template is an an HTML file where we use special markup to insert data generated by the application, in this case, by the `DApp.py` file above. 

```python
<!DOCTYPE html>
<html>
<head>
  <title>FunnyToken - Join our Fun ICO!</title>
  <link href='https://fonts.googleapis.com/css?family=Open+Sans:400,700' rel='stylesheet' type='text/css'>
    <script src="https://ajax.googleapis.com/ajax/libs/jquery/1.5.2/jquery.min.js"></script>
  <script type="text/javascript" src="/static/js/jquery.qrcode.min.js"></script>
</head>
<body>
  <h1>FunnyToken</h1>
  <h2>The Funniest ICO Ever!</h2>
  <h3>You have <span class="black">{{balance}}<span id="balance"></span> Funny</span></h3>

  <br>
  <h1>Buy FunnyToken!</h1>
  <p>Send Ether to this Address:</p>
  <div id="qrcode"></div>


  <br><label for="receiver">Your Address:</label><input type="text" id="receiver" placeholder="{{account}}"></input>
  <br><br>
  <span id="status"></span>
  <br>
  <span class="hint"><strong>Hint:</strong> open the browser developer console to view any errors and warnings.</span>
  <script type="application/javascript">
    jQuery('#qrcode').qrcode({text:"{{contract_address}}"});
  </script>
</body>
</html>
```
The template will reside in a `templates` directory in the same directory as our `DApp.py`. we will also need a static directory which will serve other types of files such as CSS, and JS. Notice, that we are using a [javascript library](https://github.com/jeromeetienne/jquery-qrcode) to render QR codes.

## Supporting both Development, Testing and Production blockchains
One thing web developers learn very quickly is how to maintain different configurations for both the development and production environments. It is very important to maintain function configuration for the development environments even after we go into production with our web app because we will always need a safe environment in which to test new feature without the risk of breaking the public website.

In our ICO DApp project, we will have three environments to cater to: the development, the Testnet (Ropsten), and the Main Ethereum network (mainnet). On top of that, we have two environments for running the web app: local development machine and production server. That gives us 6 different configuration sets to maintain.

Regarding the Blockchain configurations, we need to configure our ethereum providers using web3, luckily it is not much different than what we've already done to connect to our local private blockchain.

With [Web3.py](https://web3py.readthedocs.io/en/stable/node.html?highlight=infura), we have builtin support to help us connect to **Infura**. Then we will have the issue of connecting the wallet we created with Metamask to our frontend in case we want to operate from the same wallet. `Web3.py` can [help us here](https://web3py.readthedocs.io/en/stable/troubleshooting.html#use-metamask-accounts) as well. But keep in mind that this is not necessary, because it's very easy to use `Web3.py` to create a new account and then transfer Ether from our Metamask account to that new account. 

To configure Infura as a Ethereum provider we can use the built-in [Infura auto-initializer](https://web3py.readthedocs.io/en/stable/providers.html#auto-initialization-provider-shortcuts). First we need to make sure we have an environment variable  with our Infura API key. This can be simply done on a terminal:

```bash
$ export INFURA_API_KEY=YourApiKey
```
Then everything should just work from within your Flask app
```python
from web3.auto.infura import w3
# Check connection
try: 
    assert w3.isConnected()
except AssertionError:
    print("Connection to Infura node failed.")
```

### Managing Development and Production settings in Flask

