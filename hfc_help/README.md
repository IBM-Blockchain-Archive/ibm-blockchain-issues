#How to use HFC with Bluemix networks

The goal is to run the HFC's unit test `chain-tests.js` which will deploy example02 chaincode and query/invoke it.

1. Install HFC v0.5.0:

	```
	> npm install hfc@0.5.0
	```

1. Browse to `./node_modules/hfc` (this will be referred to as the "root" of the hfc module)

1. Open `./test/unit/test-util.js` and look for what you are setting as the "key value store" ie `chain.setKeyValStore(hfc.newFileKeyValStore('/tmp/keyValStore'));`
	1. Find your key value store locally and make sure the path to the folder exists
		- (in windows default is) C:\tmp\keyValStore
	1. Also delete anything in this `keyValStore` folder (you should empty this out anytime you connect to a new network or switch users)

1. Get your certificate. Go [download](https://blockchain-certs.mybluemix.net/) the one for your network. Most everyone will want to download `us.blockchain.ibm.com.cert`. People using a "High Secuirty Network" should grab the `zone.blockchain.ibm.com.cert`.
	- copy it to `./` (the root of hfc module, where package.json exists)
	- **double check** that the cert has a blank new line at the end of the cert

1. Open `./test/unit/test-util.js`
	- change `tlsca.cert` to to the name of the cert you just downloaded its in **two places**
	- next find and change lines 36-37 from:

	```
		chain.setMemberServicesUrl("grpcs://localhost:50051", {pem:pem, hostnameOverride:'tlsca'});
		chain.addPeer("grpcs://localhost:30303", {pem:pem, hostnameOverride:'tlsca'});
	```

	to

	```
		chain.setMemberServicesUrl("grpcs://<YOUR CA DISCOVERY URL HERE>:<PORT HERE>", {pem:pem});
		chain.addPeer("grpcs://<YOUR PEER DISCOVERY URL HERE>:<PORT HERE>", {pem:pem});
	```
	- check that you used the correct ports above 
	- check that you didn't mix up the ca with the peer url 
	- check the ports again 
	- check that you are using `grpcs://` not `grpc://`
	- check the ports again 
	- double check that you have `chain.setECDSAModeForGRPC(true);` (test-util.js line 33) 

1. Open `./test/unit/chain-tests.js`
	- add `chain.setDeployWaitTime(80);` to line 32 below the `var chain`
	- change `account: "bank_a"` to `account: "group1"` on **line 87**
	- leave `affiliation: "00001"` as is
	- set WebAppAdmin's password in chain-test.js **line 125**
	- add `certificatePath: "/certs/blockchain-cert.pem"` to **line 225** (also add the trailing comma to the `args` line directly above)

1. Create `$GOPATH/src/github.com/chaincode_example02` folder
	- copy `chaincode_example02.go` from this repo's help folder to the folder you just created

1. Go download/copy that **same** certificate in step 4 again.
	- copy it to `$GOPATH/src/github.com/chaincode_example02`
	- rename it to `certificate.pem`

1. Copy the `vendor.7z` file in this repo's help folder to `$GOPATH/src/github.com/chaincode_example02` and **unzip**
	- delete `vendor.7z`

1. From root of hfc module run:
	
```
	> set DEBUG=hfc
	> node ./test/unit/chain-tests.js
```

Done!

Success looks like:

```
> 1..13
> # tests 13
> # pass 13
>
> ok
```

***

#Tips
- if you get 11 pass 2 failed, the sdk didn't wait enough time for deploy to complete. increase `chain.setDeployWaitTime(80);`
- if you get a handshake error, try a different `grpc` version
- if you cannot npm install the hfc module, remove sleep dependency in package.json and delete node_modules/sleep if it exists
- to run the typescript compiler manually, type "tsc" from hfc root directory 
