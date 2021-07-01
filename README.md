# Stellar Turing Signing Server Reference Implementation

This TSS reference implementation employs two serverless services. Cloudflare workers and an AWS lambda function. The reason for this is that txFunctions are by their nature unsafe arbitrary Javascript functions. Cloudflare doesn't allow the execution of such functions thus we're splitting the workload between the much more performant and affordable Cloudflare workers and an AWS lambda function which will serve as our txFunction execution environment.

See below for specific instructions for setting up and running both services.

## Wrangler (Cloudflare)
If you haven't already go ahead and [signup for Cloudflare workers](https://dash.cloudflare.com/). You can attempt to run on their free tier but I highly suggest just biting the very affordable bullet and upgrading to their $5/mo plan which will allow you to scale much more nicely.

Next you should modify the `wrangler.toml` file to update hard coded values with your own.

1. For the `account_id` go to the workers page on [dash.cloudflare.com](https://dash.cloudflare.com) and copy your `Account ID`.
2. This Step is optional, however, if you are attempting to run more than one turret making them identifiable is the next step. To do so, change the name from tss-wrangler to another name of your choice. 
3. For `kv_namespaces` create three new kv namespaces via the wrangler cli:
```
$ npx wrangler kv:namespace create "META"
$ npx wrangler kv:namespace create "TX_FUNCTIONS"
$ npx wrangler kv:namespace create "TX_FEES"
```
Each of those commands will spit out the object you should use to replace the existing values in the `wrangler.toml` file. The values that are to be changed are the id values which can be seen below.
```
kv_namespaces = [
   { binding = "META", id = "1959d98949ba4c8aa9c333b836cdc5ca" },
   { binding = "TX_FUNCTIONS", id = "35d9a95d987b449fa44263f42d366429" },
   { binding = "TX_FEES", id = "6881d21a900540d9b36aa5c6f9465ea5" },
]
```
4. Finally for `vars` set `STELLAR_NETWORK` to either `TESTNET` or `PUBLIC` to toggle this Turret between using either the Test or Public Stellar network passphrases. For `HORIZON_URL` place in the url for the horizon service your Turret will consume. This should match with either the Test or Public network passphrase which you just set for the `STELLAR_NETWORK` variable. For `TURRET_ADDRESS` just use any valid, funded, Stellar account you privately own. This is the account into which fees will be paid as txFunctions are uploaded and run on your Turret. Next set `TURRET_RUN_URL` to `null` for now until we've got the Serverless AWS lambda setup with it's endpoint, at which point you'll update this value to that url. Finally set the `TX_FUNCTION_FEE_DAYS_TTL`, `XLM_FEE_MIN`, `XLM_FEE_MAX`, `UPLOAD_DIVISOR`, and  `RUN_DIVISOR` values to reasonable defaults. (For more info on these fee variables review the [Fee Wiki](https://github.com/tyvdh/stellar-tss/wiki/fees).)

Now that the `wrangler.toml` file has been updated let's move to the `stellar.toml` file. This file is where you'll create your Turret's `stellar.toml` file particularly noting the `[TSS].TURRETS` array. This will be an array of other Turret addresses that you trust to cohost txFunctions with in the case of txFunction healing. For now just make sure to include your own `TURRET_ADDRESS` which you selected in the previous steps, replacing the address on the line noted with ```#self```.

Once you've got that go ahead and upload it to the `META` kv store you instantiated earlier.
```
$ npx wrangler kv:key put --binding=META "STELLAR_TOML" ./stellar.toml --path
```
Make sure to run these wrangler commands from the `./wrangler` directory

Finally to deploy the project run the following commands within the `./wrangler` directory:
```
$ npm i
$ npm run deploy
```

You may have to work through a few errors to get logged into your Cloudflare account but the wrangler cli errors are typically quite helpful. Feel free to update this README with more clear instructions as it's been ages since I started from scratch on my first wrangler project.

5. Once you've successfully got your project created and running upload a `TURRET_SIGNER` Stellar secret key to your Cloudflare worker.
```
$ npx wrangler secret put TURRET_SIGNER
```
When the dialog asks your for a value paste in a valid Stellar **secret key**. Most often this will be the secret key counterpart to your `TURRET_ADDRESS` but this isn't a requirement. This key is used to authenticate requests between your Cloudflare and Serverless services, nothing else.

## Serverless (AWS)
Next we have the Serverless lambda endpoint which is hosted with AWS but deployed using the far more sane [serverless.com](https://serverless.com) cli tool. If you haven't go create both an [AWS console account](https://www.amazon.com/) and a [serverless.com account](https://www.serverless.com/dashboard/). It is important that once you have both of these accounts that the AWS is linked as a provider with the serverless.com account. Once you have those setup ensure you've got the [serverless cli installed](https://github.com/serverless/components#quick-start).

There's only two things you'll need to update in this repo in the `serverless.yml` file. The `provider.environment.turretBaseUrl` should be replaced with the worker base url which your wrangler service is hosted on. The `provider.environment.turretSigner` should be replaced with your Turret's `TURRET_SIGNER` **public key** set in the Wrangler setup. This connection is what secures and protects access between the Cloudflare and Serverless APIs. Remember Cloudflare gets the **private key** and Serverless gets the **public key**.

Before running the following commands, it is importnat that a new serverless.com application is set up and the values entered at the top of the `serverless.yml` file. Selecting serverless framework from the list of options and then entering in the desired names for app and service, is all that is required on the serverless side of things. Take all of these values and enter them into the top of the `serverless.yml` file.

Now it'll be the fun task of getting:
```
$ npm i
$ npm run deploy
```
To successfully run from within the `./serverless` directory. Follow any errors carefully and you should be able to get successfully deployed pretty quickly. The most probable issue will be you need to manually create an app in the [Serverless dashboard](https://app.serverless.com/tyvdh) and attach some new IAM credentials to it manually. There's a helpful UI walk through they have so you should be able to sort it out. Again feel free to update these docs with more clear instructions as you sort out the nuances of setting the Serverless service up.

When you finally get success on this task you'll be rewarded with an endpoint where your function is hosted. Copy that base url and paste it as the value for the `TURRET_RUN_URL` `var` back in the `wrangler.toml` file in the `./wrangler` directory. Don't forget to redeploy that project after this update by using `$ npm run deploy` in the `./wrangler` directory.

## Congrats!
Assuming both `npm run deploy`'s are now firing off without a hitch you should have a fully functional Turing Signing Server ready to participate in the TSS network delivering decentralized smart contracting functionality to anyone and everyone who chooses to use your Turret. Nice!

## [API Docs](https://github.com/tyvdh/stellar-tss/wiki)

## Disclaimer
This is alpha software representing a reference implementation for the [TSS protocol](https://tss.stellar.org/).

For this reason I strongly suggest either:  
A) Leaving your `STELLAR_NETWORK` set to `TESTNET`  
or B) Encouraging users to leave themselves as a majority signer on any controlled account they're attaching Turret signers to
