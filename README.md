********************************************

## **Haskell Validator and Police Scripts**


### **Tangent NFT Staking**

********************************************

Readme deprecated, needs update

********************************************

Inside the Nix-Shell of the Plutus Apps folder:

Compile with:
> cabal build

They can also be installed to access executables without using cabal.
> cabal install

********************************************
   
**Smart Contracts**
  
I would like to start by explaining a little how Smart contracts work.
  
Smart contracts are written in Haskell and this is what we call the on chain part.
  
Code written in Haskell is converted by the Haskell compiler into Plutus Script code. This Plutus Script code is reduced by means of a hash function to a specific and unique hash and address within the blockchain. Any change to the Haskell code implies a new hash and a new address. To send ADA or Tokens to a contract, that is, to the address that we obtained from its hash, it is as simple as when ADA or Tokens are sent from one wallet to another wallet. The receiving wallet does not intervene in any decision. In the same way, neither the contract nor its logic intervene when receiving.
  
The contract logic is executed when you want to consume a uTxO that is at the address of the contract. This is similar to when I want to consume a uTxO that is at some address in my wallet, and for that I need the private key of that wallet.
  
The Plutus Script code of the contract is what would be the private Key needed to consume a uTxO that is in my wallet, only a particular Haskell code, when reduced to its hash, matches the hash of a particular address.
  
The Plutus code, which contains the logic of the contract, is sent to the blockchain at the moment in which an address of that contract is to be consumed. It will first verify that the hash of that Plutus is the one that corresponds to that address and then the received Plutus code will be executed and according to its output, a true or false Boolean, the consumption of that uTxO will be approved. Before the last Vasil update, it was necessary to send the same contract over and over again, every time you wanted to consume some uTxO from his address. From the Vasil fork it is possible to send the code only once and then in future txs only refer to the tx where the code is already hosted on the Blockchain. This considerably reduces the size of the txs that want to consume uTxO from a contract.
  
**Values**
  
Each uTxO (unspent transaction output), which are the outputs of the tx (transfers), have a value. That value is a sum of ADA and tokens. From now on I will call it the value of a uTxO.
  
**Datums**
  
There is a difference between the uTxO that are in the address of a wallet with those that are in the address of a contract. Those for contracts have a piece of data attached to them and that piece of data was arbitrarily created at the time ADA or tokens were sent to the contract address. This data is called Datum and it is where information is stored that the contract will take into account when validating the consumption of a uTxO and sending its values to another address in the form of a new uTxO.
  
For example, in a locker contract for a defined time, there is a deadline in the Datum, a date that is determined when creating the transaction and is sent together with the values that are to be locked in the contract. This date will be reviewed by the contract at the time someone wants to consume that uTxO, to withdraw the funds from the contract, and will approve or not the extraction.

**Passivity**

Perhaps you have already been able to observe something fundamental about smart contracts: the passive nature of contracts.
The contracts themselves are not executed by themselves, but only and only when someone initiates a transfer that wants to consume a uTxO that is in a contract, and its only function is to approve or not this transaction.

**Immutability**

Another important issue to take into account in the operation of contracts, Datums and uTxOs, is that a Datum once created cannot be changed, it is immutable. This means that if I have to update a Datum, I need to consume the uTxO where the Datum I want to update is located and create a new uTxO with the new Datum. Of course, if there were ADA or Tokens in the uTxO that I consumed, logically I have to create the new uTxO with that same amount of ADA or Tokens. In any case, these are some of the basic verifications that the contract will have to do when allowing the update of a Datum: that it check that the Datum consumed and the one created are respecting certain business logic and that the values of the uTxO also correspond.

**Simple staking: centralized in a single uTxO**

Taking into account the aforementioned, I am going to first define a staking system in the simplest way possible. At the address of the contract there is a single uTxO with a Datum that contains all the information of the Pool, who created it, I will call these users masters, their deadline, the date the pool ends, the interest rate they pay , if it charges fees, and the registered users, together with the funds that the users invested and the rewards that they were collecting. This system is unified and all the important information is in a single uTxO, whose value is the sum of all the funds provided by the masters, the investment of all the users minus the sum of all the rewards claimed.

The first step is a Master user depositing funds into the contract. To do this, consume the uTxO with the Datum of the Pool, send ADA or tokens from your own wallet and generate a new output with a new Datum, where your fund appears, and whose value is the one you added from your wallet. This value will remain in this uTxO available for the payment of rewards of registered users. At the end of the Pool, the master will be able to recover the ADA or Tokens that have not been claimed by the users in the form of rewards.

In the case of registering a new user within the pool, I have to consume the uTxO with the Pool's Datum and create a new one whose Datum is updated, reflecting the new user and his investment. The value in ADA or Tokens of the output will have to be equal to the amount consumed, the previous one, plus the new funds that the user added with his investment from his own wallet.

When wanting to obtain rewards, the user creates a tx that consumes the uTxO with the Pool Datum, and creates an output with the updated Datum, reflecting the new rewards claim. In this case, there will also be an output to the user's wallet with the payment of the rewards. The final amount in the uTxO that remains in the contract will be the same as before minus the rewards payment made.

As we already mentioned, each time that uTxO is consumed to update the Datum of the Pool type, the contract will be executed and will validate that consumption or not: for this it will verify what type of action is being carried out and accordingly what type of modification it allows in the Datum and how the values of the uTxO are affected, if they should be kept the same, if they should grow (thanks to a new fund from a master or a new investment from a user) or if they should go down (thanks to a claim of rewards of a user). It will only approve that tx that is doing a correct action, a correct modification of the Datum and of the values of the uTxO.

**Security problem**

As we mentioned before, there is no validation or logic that is executed when adding a uTxO to the address of a contract. So nothing prevents a malicious user from creating a uTxO with a Datum similar to the valid Datum of the Pool, but with some difference that represents some benefit. For example, if a user's rewards are based on the date of their registration, or the date of their last rewards claim, they could create a Datum that has an earlier date than the real one in their registration, with the purpose of so that you can request more rewards than what you would be entitled to and the contract would give them to you.

So, how can the contract know which uTxO has a valid Datum and which does not? For this, NFTs are used.

**NFT**

The only way to verify which uTxO are valid and which are not is by using NFT. By definition, an NFT is unique and unrepeatable, so we can be sure when we look for a uTxO that the one with the appropriate NFT will be the only valid one. If that NFT accompanies a uTxO with a Pool type Datum, for example, and I want to update that Datum, the output has to be another uTxO, with the updated Datum and the same NFT.

The NFT must be minted when creating the first uTxO with that type of Datum and must be burned when finally consuming that uTxO, in our case, when a Pool closes. While the Pool is active, this NFT goes from uTxO to uTxO as the Datum is updated, indicating which uTxO is the valid one among all the uTxO that may exist in the direction of the contract.

**Concurrency issues**

The previous staking model may work for theoretical purposes, but in practice it may suffer from a concurrency problem: if all the Pool information is in a single uTxO, whose Datum must be updated every time there is an operation in the Pool, then it is very possible that two or more users want to register or obtain rewards at the same moment and only one of them will be able to consume that uTxO at that moment. Only the first to send the tx to the blockchain can be validated. The rest will generate txs that will be invalid and not approved despite having created a valid tx at the time. In the best of cases, they will have to wait for the previous tx to be executed, for a new uTxO to be generated with the Datum of the

**Problemas de concurrencia**

The previous staking model may work for theoretical purposes, but in practice it may suffer from a concurrency problem: if all the Pool information is in a single uTxO, whose Datum must be updated every time there is an operation in the Pool, then it is very possible that two or more users want to register or obtain rewards at the same moment and only one of them will be able to consume that uTxO at that moment. Only the first to send the tx to the blockchain can be validated. The rest will generate txs that will be invalid and not approved despite having created a valid tx at the time. In the best case, they will have to wait for the previous tx to be executed, for a new uTxO to be generated with the Pool Datum, and create a new tx that will consume that new uTxO. In the worst case, your funds will not be affected, but you will be unable to create tx to register or get rewards.

** Parameterized contracts **

I also want to mention another possibility that contracts allow: when executing and validating a tx, they can not only access the Datums if they cannot make use of some parameters. These parameters must be indicated when creating the contract and will be hard-coded in the Plutus code and will also be part of the hash and its address. This means that the same contract code, written in Haskell, that receives different parameters, will generate a different Plutus code, therefore, a different hash and address as well.

In our case, it is not necessary to change the Haskell code of the staking contract every time I want to modify any of its parameters. I simply use the new parameters, the same code, and get a new contract address, which will make use of those new parameters indicated. For example, change the list of Masters or the interest rate paid by the contract.

**EndPoints**

The endpoints of a contract are what we call the OffChain code of the contract. This code is also written in Haskell but is not compiled or converted to Plutus Script nor does it run on the blockchain. It is code that is compiled and executed on the computer or server that will provide users with interaction with the contract.

In many cases it is the code that will create the txs that will later be sent to the blockchain for validation by the contract. It serves as an interface so that different frontends have a common access to the contract and transparent in terms of the assembly of the tx. They should only send parameters and the OffChain part will be in charge of assembling the correct txs.

It is important to take into account that the OffChain code and the OnChain code can have code in common, that means that if the OnChain contract uses a function to validate, for example, the amount of rewards that a user can obtain in a staking contract, the part OffChain can also access that same function and build txs that claim that amount of rewards, calculated in the same way, making sure that the OnChain part will then approve that tx.

**Staking Plus: solution to the concurrency problem**

In this case there will not be a single uTxO that needs to be consumed over and over again for each user action.
I will have several uTxO, and each user will be able to interact with one of them, thus dividing the transit and consumption.

The users of the contract will also be two, the master users and the normal users, which I will only call masters and users respectively.

The contract is of the parameterized type, which means that I must first create the parameters of the Pool, attach them to the Haskell code and from there obtain a Plutus code that makes use of those parameters, and from that code I obtain a hash and a particular address . Let's remember that these parameters cannot be changed afterwards, they are hard-coded in the contract code that is in that address.

**Pool Parameters:**

```
{
ppPoolID::PoolID,
ppPoolIDTxOutRef::LedgerApiV1.TxOutRef,
ppMasters :: Masters,
ppDeadline::Deadline,
ppInterest :: Interest ,
ppMinumInvest :: Invest,
ppMaximunInvest::Invest,
ppFundIDPolicyId::LedgerApiV1.CurrencySymbol,
ppNFTUserIDPolicyId::LedgerApiV1.CurrencySymbol,
ppValidTimeRange::LedgerApiV1.POSIXTime,
ppMinimumClaim :: Proffit
}
```

Some clarifications:
**ppPoolID**: the NFT that identifies this Pool.
**ppMasters**: Pool Masters: a list of Payment Pub Key Hashes that determine which wallets may have privilege functions over the Pool.
**ppDeadline**: Date on which the Pool ends.
**ppInterest**: For the calculation of rewards.
**ppFundIDPolicyId**: I save the policy id to control if the NFTs of new FundDatums are correct.
**ppNFTUserIDPolicyId**: I save the policy id to control if the NFTs of new UserDatums are correct.

We will also have **three types of valid Datums** in the different uTxOs that will be at the contract address: **Pool Datums, Fund Datums and User Datums**.

**There will be only one valid uTxO with a Datum of the Pool type** and it will have a unique NFT called Pool ID in its value. The contract will then be able to verify the validity of a uTxO with a Pool Datum if and only if it has in its value the NFT Pool ID that corresponds to the Pool ID indicated in the contract parameters.

If I change the Pool ID from the contract parameters, then I have a new contract address. Therefore, in a single contract address there is only one possible Pool ID. That is why I can look for that Pool ID in the values of the uTxOs and validate the only uTxO that has it. In this way, no one can create a uTxO with a Pool Datum equal to the valid one at that contract address. There is only one NFT once and in this case it identifies a single uTxO and no one can impersonate it.

**The only field in the Pool Datum is:**

```
{
pdFundID_TNs::[FundID]
}
```
There will be a single uTxO in the contract with a Pool Datum but **there may be many uTxO with Fund Datums.**

The pdFundID_TNs list of Fund IDs declared above will indicate the validity of the Fund Datums. Only those uTxOs with Fund Datums with any of the NFT Fund IDs defined in this list will be considered valid.

The way to corroborate the validity of these uTxOs is that they must have in their value an NFT called Fund ID that will be different and unique for each one of them. Every time I want to create a new uTxO with a Fund Datum I must minte a new NFT of the Fund ID type, I must create a Fund Datum and I must update the Pool Datum and add the new Fund ID to the list of Fund IDs.

To update the only Pool Datum that will exist in the contract, I must consume the uTxO that owns it and this is where the contract logic is executed. This is where I make sure that the creation of a new Fund Datum is verified by the contract. The contract is the one who decides if everything is correct: if a Fund Datum is being created with the correct data, if it has a unique NFT called Fund ID, if a new updated Pool Datum is being created: if it is being added to the Pool Datum old the new Fund ID mentioned to the list of Fund IDs and, last but not least, if that NFT is mined with a minting policy that ensures it is an NFT. This is why the ppFundIDPolicyId field of the Pool parameters is used.

If it were not an NFT, it could happen that I create a valid Fund Datum, with a Fund ID, and this is registered within the Pool Datum, within the list of Fund IDs. Then a malicious user could come and create a new Fund Datum, with values that benefit him, and with the same Fund ID mined and added. The only way to prevent this from happening is to make sure the mined Fund ID is an NFT. The only way to know that it is an NFT is to know with what policy it was mined. That's why I'm going to create a policy for NFT mining and I'm going to declare it in the Pool parameters. Then, in your code, you can control that the Fund ID is a minted NFT with that particular policy.

As we said, there will then be as many uTxOs with Fund Datums as you need, they will be valid as long as your NFT Fund ID is in the list of FundIDs of the Pool Datum. Each Fund Datum is one more channel that opens the bandwidth of our contract. Users will interact with one or the other Fund Datums and will be able to do so in parallel, avoiding the aforementioned concurrency problems.
  
**The Fund Datum fields will be:**

```
{
fdFundID_TN::FundID,
fdMasterFunders::[MasterFunder],
fdUserID_TNs::[UserID],
fdCashedOut::Proffit
}
```
  
**fdFundID_TN**: The name of the NFT that identifies this uTxO with this Fund Datum
**fdMasterFunders**: A list of MastersFunders, which determines which Masters put how much into the Pool as funds for users to collect their rewards. These funds will be as securities in the uTxO with this Fund Datum.
**fdUserID_TNs**: just like the list we saw before in the Pool Datum, of Fund IDs, which is used to validate uTxOs with Fund Datums, here is a list of User IDs that will be used to validate uTxOs with User Datums.
**fdCashedOut**: How much has been claimed in the form of rewards from this uTxO with Fund Datum.
  
To register a user in the Pool, I must update any Fund Datum of any uTxO of the contract. I will have to mine an NFT User ID, add it to the list of User IDs of the Fund Datum that I am up to date, and create a User Datum where I record the user's data and their investment. Again, this is where the logic of the contract intervenes. It will be executed because you want to consume a uTxO that is in your address. You want to consume a uTxO with a Fund Datum because you want to update it and add the new User ID to the list. The contract will approve the tx only if, among other things: a valid User Datum is being output: whose registration date is correct, whose investment amount corresponds to the values in the uTxO that comes from the user's wallet and that these values are in this output uTxO. That this output uTxO with User Datum has an NFT of the User ID type minted with the correct policy. That a uTxO of the contract is being consumed with a Datum of the Fund Datum type. That an output is being generated with a Datum of the Fund Datum type. That the new User ID is being added to that Datum. That the values of the uTxO that the Fund Datum possessed are not being modified in any way.
  
If all this is correct, a uTxO will be created in the contract with a valid User Datum. **There will be as many uTxO with User Datum as registered users within the Pool**.

**User Datum fields are**:
```
{
udFundID::FundID,
--where is it registered
udUserID_TN::UserID,
--record id, of the userDatum that represents your record
udUser :: User,
--pkh of the user, to verify claims
udInvest :: Invest,
udCreatedAt::LedgerApiV1.POSIXTime,
udCashedOut::Proffit,
udRewardsNotClaimed::Proffit,
udLastClaimAt::Maybe LedgerApiV1.POSIXTime
}
```
This is where we can see how important it is to know which uTxOs are valid and which are not. If a malicious user created a uTxO in the contract with a User Datum that says they registered two years ago, they could then get rewards for that entire period and I would have no way to verify the validity of those dates.
Anyone can write Datums with any shape in a contract address.
I have to have the mechanisms to know how to differentiate the valid from the not.
That is what NFTs are used for and this chain of links that could be summarized here:

An NFT Pool ID is defined and assigned in the contract parameters.
The contract parameters cannot be changed and together with the Plutus code they determine a unique address.
At that address there is only one possible Pool ID and one and only one uTxO may have that NFT in its value.
That uTxO with the Pool ID will contain the Pool Datum that we will call valid.
In that Pool Datum will be the list of valid Fund IDs.
Any uTxO with a valid Fund ID will contain a valid Fund Datum.
These Fund Datums detail the list of registered valid User IDs.
All uTxOs that have a valid User ID will contain a valid User Datum.

In this way the contract can at any time know if a User, Fund or Pool Datum is valid.

**Staking Plus Contract EndPoints**

**1. Master Create Pool**:

The first endpoint is the one that will create the Pool Datum, where Fund IDs and User IDs can then be registered. Here the contract is not executed, since no uTxO of the contract is being consumed.

The NFT Pool ID that corresponds to the contract parameters must be minted.
An NFT Fund ID must be minted with the correct policy specified in the contract at ppFundIDPolicyId.
A Pool Datum must be created with the Fund ID in the Fund IDs list.
A Fund Datum must be created indicating the fund of values that the master deposits.

A tx must be created whose entries are:
Pool ID and Fund ID mining.
The values of the master for the fund, which come out of your own wallet.
And the outputs:
A uTxO with a Pool Datum and the NFT Pool ID.
A uTxO with a Fund Datum, the NFT Fund ID, and the master values for the fund.
  
**2. Master Fund Pool:**

Here the uTxO with the Pool Datum is consumed to create a new Pool Datum with the new Fund ID. A new uTxO is also created with a new Fund Datum.
  
A tx must be created whose entries are:
The mining of a new Fund ID.
The consumption of the uTxO with valid Pool Datum and the NFT Pool ID.
The values that the master arranges for the new fund, which come out of his own wallet.
And the outputs are:
A uTxO with an updated Pool Datum, with the new registered Fund ID, and in its value the NFT Pool ID.
A uTxO with a new Fund Datum, the new Fund ID, and the master values for the fund.
  
**3. Master Fund & Merge:**

Here the uTxO with the valid Pool Datum is consumed, and one or several uTxOs with valid Fund Datum are consumed, to have as output a new uTxO with Pool Datum and a single new uTxO with Fund Datum. It serves to unify uTxO with Fund Datums.

A tx must be created whose entries are:
The mining of a new Fund ID.
The consumption of the uTxO with valid Pool Datum and the NFT Pool ID.
The values that the master arranges for the new fund, which come out of his own wallet.
The consumption of one or more uTxO with Fund Datums and their respective Fund IDs.
And the outputs are:
The burning of the Fund IDs that I am joining.
A uTxO with an updated Pool Datum, with the new registered Fund ID, and in its value the NFT Pool ID.
A uTxO with a new Fund Datum, the new Fund ID, the values of the master for the fund and the sum of all the values that were in the uTxOs with Fund Datums that I am joining.
  
**4. User Registration:**

Some uTxO with Fund Datum is consumed and a new uTxO is obtained with a Fund Datum where the User ID registered in the list of User IDs appears and a new uTxO with a User Datum together with the recently minted NFT User ID. The values of that uTxO with Fund Datum must not be touched. The user registry must update the Datum but not modify the values of that uTxO.

A tx must be created whose entries are:
Mining of a new User ID.
The consumption of the uTxO with Fund Datum and the NFT Fund ID.
The values that the user has for the investment, which come out of his own wallet.
And the outputs are:
A uTxO with an updated Fund Datum, with the new registered User ID, and the values and NFT Fund ID that were previously present in the uTxO with Fund Datum. Those values are not touched. Only the Datum is touched. It updates.
A uTxO with a new User Datum, whose value contains the new NFT User ID and the values that the user provided from his wallet.
  
Something to pay attention here: to avoid concurrency problems, when registering the user it is not necessary to consume the uTxO with the Pool Datum. Only some of the uTxO with Fund Datums. And there can be many of them, therefore, our bottleneck will be as small or as big as we want.

**5. User Get Rewards:**

The uTxO with Fund Datum where the user is registered is consumed, and a new uTxO is obtained from the output with a Fund Datum where the new amount claimed appears. From this uTxO that is consumed, the values are taken to pay the user their reward.

The uTxO where the User Datum is is also consumed to reflect the date of this claim and use that date in the future calculation of next rewards. The values of that uTxO with the User Datum must not be modified. The user's investment passes from input to output without touching.

A tx must be created whose entries are:
The consumption of the uTxO with Fund Datum and the NFT Fund ID, and the values available to pay rewards.
The uTxO with the User Datum, which must be updated with the new payment.
And the outputs are:
A uTxO with an updated Fund Datum, indicating the funds claimed, and whose value is the same as before minus what was claimed.
A uTxO with updated User Datum, whose value is not modified.
An exit to the user's wallet with the claimed rewards.
  
Something to pay attention to here: to avoid concurrency problems, when making user rewards claims, it is not necessary to consume the uTxO with the Pool Datum. Only the uTxO with Fund Datum where the registered user appears. Here arises the need to take into account: if the uTxO with the Fund Datum where the user is registered runs out of funds, it must be possible for another uTxO with Fund Datum to be attached, where the user is not registered, but there are funds, together with the Fund Datum where it is registered, and with these two uTxOs the contract can validate the user's registration and pay the rewards. As will be explained later, this opens a security gap, and that is why in this type of tx, when you want to use more than one uTxO with Fund Datums, you attach the uTxO with Pool Datum, in order to verify the Fund Datums that I am using. Read below problems and their solutions.

***6. User Invest Rewards:**

It is a double action. Rewards are claimed and used to increase the funds invested by the user. Rewards are not sent to the user's wallet but are sent back to the contract.

**7. User Recover Investment:**

The user recovers in his wallet the funds that he put in the investment.

**8. Master Recover Funds:**

The master recovers in his wallet a proportional amount of what he put in, of the funds that have not been used to pay the rewards.

**9. Master Close Pool:**

The Pool is closed. All masters are sent the funds that correspond to them from what is left over. Their investments are sent to all users.

**Problems and their solutions**

I am going to expand a little more on how uTxO can be created with false Datums in the direction of the contract and how they can be validated.

If in each tx that I create, I attach and consume the uTxO with the Pool Datum, the uTxO with the Fund Datum and the uTxO with the User Datum, there would be no problem to validate them all in a simple and staggered way. The uTxO with Pool Datum is valid if it has the NFT Pool ID that matches the Pool ID of the contract parameters. The uTxO with Fund Datum is valid if it has an NFT Fund ID present in the list of Fund IDs of the previously validated Pool Datum. And the uTxO with User Datum is valid if it has a previously validated NFT User ID registered in the list of User IDs of the Fund Datum. In this way, I am sure that I am dealing with uTxOs with valid Datums.

But this is not always the case. In fact, if that were the case, the aforementioned concurrency problem would appear again: if every action in the Pool must consume the uTxO with the Pool Datum, only one can consume it at a given moment. That is why I must have mechanisms so that the Pool can work without the need to consume the uTxO with Pool Datum all the time.

A solution that seems possible to me is to use what is called Inline Datum, another new functionality from the Vasil era, where when creating a tx you can refer to an existing Datum, without having to consume the uTxO where it is present. The contract could then have access to that Datum, in this case to our Pool Datum. In principle, this may work and perhaps provide some extra security possibility, but it would not be enough, since in order to validate that Datum I have to access the value of that uTxO and verify that it has the necessary NFT Pool ID. This I have not tried yet.

So, as we have been saying, if a tx wants to consume only a Fund Datum and a User Datum, I cannot verify it, because I do not have access to the valid Pool Datum. I also cannot know if the Fund Datum is valid, even if I have a minted NFT Fund ID with the correct policy. I will explain this a bit more.

A minting policy can be used by anyone. It is a code that may be available. In fact, it must be available so that an audit can see what it does and validate, and thus ensure that what it produces is NFT, for example. In addition, our code is intended to be open source, therefore, the policy we use can be used by anyone to mint their own NFTs.

To minte NFT Pool ID and NFT Fund IDs, you can make the minting policy use extra validation on which wallets can mint. And in that case I can limit that minting to the Masters of the Pool. That way I make sure that the Pool masters can only make txs that minte using that policy. This is done using a parameterized minting policy.

As explained with the Smart Contracts, the same minting policy is defined with specific parameters, in this case a list of Payment Pub Key Hashes of the masters, and these parameters are hard-coded in the minting policy. The sum of the Haskell code of the minting policy plus the parameters generate a Policy ID (similar to the hash of Smart Contracts) that is unique and responds to a specific code and parameters. When I want to lie with that Policy ID, I must supply the code and the parameters that generated that Hash, I cannot change them. Therefore, that Policy ID will verify that whoever uses it is one of the Masters. This adds a layer of security, but it's not enough. We must also protect the masters from a malicious master.

But to mitigate the User ID NFT I cannot limit this minting policy to some wallets, because any user should be able to create a tx where they register in the Pool. Also remember that when registering the user must supply values from his wallet, then only he can do it.

A malicious master could fake a new Fund ID, which will not be registered within the valid Pool Datum. It could then fake a new User ID and create a fake Fund Datum containing that registered User ID. You could create a fake User Datum with a fake registration date and you could send those Fake Datums to the contract. You could then pretend to be the benefited user, and make a rewards claim to the contract. To do this, it would send the Fund Datum and the User Datum that it created to the contract and obtain false benefits in its wallet, since the contract could not know that these Datums are false.

In any case, up to now there is no problem, the funds that he would receive leave the uTxO with the false Fund Datum that he himself created and funded. You would be paying with your own money. The problem arises when the funds of that uTxO are emptied because there is a mechanism for such cases, which allows users to obtain rewards using a uTxO with additional Fund Datum where there are funds. In that case, using the fake Fund Datum and Fake User Datum, you could get rewards from a valid Fund Datum.

To avoid this scenario, the mechanism requires that when a user wants to claim funds from a uTxO where they are not registered, that tx must include the uTxO with the valid Pool Datum of the Pool. Having that Pool Datum, I can verify the Fund Datums that I am receiving and check that they are valid.

Another thing that can happen is that a new user, other than the malicious one, registers in the fake Fund Datum. In this case, you could obtain rewards from that uTxO, affecting only the malicious user, but when the funds of that uTxO are emptied, they will not be able to claim funds from another Fund Datum, since for this they must include the Pool Datum and with it will verify that your registration is in a false Fund Datum. This is an unfortunate situation, but it can be avoided if the means by which a user registers are through official channels. Our FrontEnd, connected with our OffChain, will make sure to register users in valid Fund Datums. But since we cannot prevent users from interacting with our contract through other channels, these users would be at risk of signing up for fake Fund Datums. They do not run the risk of losing their funds, they will always have mechanisms to claim their investment, but they would lose the opportunity to obtain the rewards that correspond to them.
