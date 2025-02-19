# Webinar Episode 1 Code Challenge Solution

This is my solution to the code challenge at the end of the code challenge at the end of the [REAMDE file](https://github.com/trufflesuite/webinar-episode-01) for the [Truffle Webinar](https://www.crowdcast.io/e/truffle-webinar-series-episode1).


<br/>


## What We Have Already

So, from episode 1 we have a Solidity contract called `SimpleStorage` with two functions, `getStoredData` and `setStoredData` which respectively read and write some unsigned integer to and from the blockchain.

```solidity
//SPDX-License-Identifier: MIT
pragma solidity >=0.5.0 <0.8.0;

contract SimpleStorage {
    uint256 public storedData;

    function getStoredData() public view returns (uint256) {
        return storedData;
    }

    function setStoredData(uint256 x) public {
        storedData = x;
    }
}
```

We also have this nice JavaScript test file that verifies that our contract can do these things:

  - it can be compiled and deployed with no errors.
  - it has an initial value of 0 when first deployed.
  - it stores some positive integer.

Here's the full code file:

```javascript
const SimpleStorage = artifacts.require("SimpleStorage");

contract("SimpleStorage", function (accounts) {
  describe("Initial deployment", async () => {
    it("should assert true", async function () {
      await SimpleStorage.deployed();
      assert.isTrue(true);
    });

    it("was deployed and it's intial value is 0", async () => {
      // get subject
      const ssInstance = await SimpleStorage.deployed();
      // verify it starts with zero
      const storedData = await ssInstance.getStoredData.call();
      assert.equal(storedData, 0, `Initial state should be zero`);
    });
  });
  describe("Functionality", () => {
    it("should store the value 42", async () => {
      // get subject
      const ssInstance = await SimpleStorage.deployed();

      // change the subject
      await ssInstance.setStoredData(42, { from: accounts[0] });

      // verify we changed the subject
      const storedData = await ssInstance.getStoredData.call();
      assert.equal(storedData, 42, `${storedData} was not stored!`);
    });
  });
});
```

And of course we have a migration file which is used  to deploy the contract to a blockchain, either running locally or on the blockchain:
```
const SimpleStorage = artifacts.require("SimpleStorage");

module.exports = function(deployer) {
  deployer.deploy(SimpleStorage);
}
```

Okay! Now that we've gone over where we're starting from here, let's review what the code challenge wants us to do.



<br/>


## Code Challenge 

- a mapping mapping[address => uint] and update it every time someone calls setStoredData
- a function getCount(address _address) that returns the count associated with _address
- a modifier to guard that only the owner can call getCount which should revert when invoked with the wrong caller.
- This Truffle Assertion plugin which helps test for revert.


<br/>


## Thinking About The Behavior

It's important to make sure you understand the behavior before you start coding. What I think they are asking for here (and I could be wrong to be honest) is a mapping (similar to a hashmap in other languages) that is stored on the blockchain where the _keys_ of our mapping are the unique id of the user calling "setStoredData", and the values of the mapping are the number which the user has last set his or user value to (with 0 as the default).

Then we also want a "getCount" function which can only be called by the contract owner, and this function should accept a user id and return the current number associated with that user in our mapping (and use 0 as a default if the key doesn't exist in the mapping).


<br/>


## Naming Our New Contract

Naming things can be hard! It can also be hard to know sometimes whether you should make a new file or modify an existing file...

In this case, I have decided to make a new contract named `OwnerStorage`.

<br/>


## Creating New Files For Our New Contract 

Remeber, TDD means you **write the tests first!**

So, let's create a test that does the bare minimum to check that our contract exists, doesn't error, and can be deployed.

owner-storage.js

```javascript
const OwnerStorage = artifacts.require("OwnerStorage");

contract("OwnerStorage", function (accounts) {
  describe("Initial deployment", async () => {
    it("should assert true", async function () {
      await OwnerStorage.deployed();
      assert.isTrue(true);
    });
  });
```


When we run this we should see an error that "OwnerStorage" does not exist, so let's make it!

/contracts/OwnerStorage.sol
```solidity
//SPDX-License-Identifier: MIT
pragma solidity >=0.5.0 <0.8.0;

contract OwnerStorage {


}
```


## Describing The New Functionality

Okay, now let's write a test that describes our new functinality.

Here I'll just look at two different users, the first and second account of our array created by truffle.

_Keep in mind that Truffle by default makes the first user, accounts[0], the contract owner!_

<br/>

Notice that this test is extremely similar to our SimpleStorage test!

The only difference is that we are calling "setStoredData" twice, calling once as each of our users, setting the first user's number to 1 and the second user's number to 2.

Then we are reading the value for each user, and asserting that that are equal to 1 and 2, respectively.

```javascript
describe("Functionality", () => {

    it("should store a value for each user", async () => {

      // get subject
      const ownerStorage = await OwnerStorage.deployed();

      // Set the values for the first three users to 1, 2 respectively.
      await ownerStorage.setStoredData(1, { from: accounts[0] });
      await ownerStorage.setStoredData(2, { from: accounts[1] });

      // get the actual values now
      const user1storedData = await ownerStorage.getStoredData.call({ from: accounts[0] });
      const user2storedData = await ownerStorage.getStoredData.call({ from: accounts[1] });

      // verify we changed the subject
      assert.equal(user1storedData, 1, `user1storedData: ${user1storedData} was not 1!`);
      assert.equal(user2storedData, 2, `user1storedData: ${user2storedData} was not 2!`);

    });

});
```


Great! Looks like a nice test to me, so let's run it and look at the output:

```shell
 1 passing (281ms)
  1 failing

  1) Contract: OwnerStorage
       Functionality
         should store a value for each user:
     TypeError: ownerStorage.setStoredData is not a function
      at Context.<anonymous> (test/owner-storage.js:19:26)
      at processTicksAndRejections (internal/process/task_queues.js:95:5)
```


Right, I guess we'll need to add some functions into that OwnerStorage contract!

We can have functions here with the same names as SimpleStorage, `getStoredData` and `setStoredData`.

Let's just put some empty functions into our contract for now:

```solidity
function getStoredData() public view returns (uint256) {

}

function setStoredData(uint256 x) public {

}
```

Then we rerun the tests, and we can see they now fail for a new reason. (hey, it's progress!)

```
 1 passing (445ms)
  1 failing

  1) Contract: OwnerStorage
       Functionality
         should store a value for each user:
     AssertionError: user1storedData: 0 was not 1!: expected <BN: 0> to equal 1
      at Context.<anonymous> (test/owner-storage.js:27:14)
      at processTicksAndRejections (internal/process/task_queues.js:95:5)

```


Ok, this is great. We have a really good test now expecting the first user's number to be 1, and failing beacuse it got zero (since we haven't implmemented the functions yet). Notice that the initial value is not null, undefined, or some other weird thing- _it's just zero without us needing to code anything to initialize it to zero!_

Notice how if we were not doing TDD we may have rushed into adding some initially unnecessary code, but beacuse we see this already it shows  up that we don't need to. Let's all take a second and reflect on how great and amazing test-driven development is... 🤩


Also, seeing this 0 in the output just reminded we that we can and should write a test for the "zero condition" for OwnerStorage as well.

All we have to not is _not_ set anything initially! We just go right into asserting that each user's value is 0.

```javascript
it("should initialize each user's value to 0", async () => {

  // get subject
  const ownerStorage = await OwnerStorage.deployed();

  // get the current values
  const user1storedData = await ownerStorage.getStoredData.call({ from: accounts[0] });
  const user2storedData = await ownerStorage.getStoredData.call({ from: accounts[1] });

  // verify the values are zero
  assert.equal(user1storedData, 0, `user1storedData inital value: ${user1storedData} was not 0!`);
  assert.equal(user2storedData, 0, `user1storedData inital value: ${user2storedData} was not 0!`);

});
```

This one passes now, but we'll also want to make sure it continuous to pass as we code out the real implmentation for our OwnerStorage functions.

<br/>

## Creating The Mapping

Ok, _now_ we can actually get to writing the business logic of saving each user's value! 

In Solidity we have the keyword `mapping` to create our hashmap style data structure, but when defining it we need to be explicit about the type for our keys and values. 

The values can be of type uint256 (might as well be consistent with the original webinar 1 code), but what type do we use for for our user identifier? 🤔

A key thing to remember is that in Solidity we have access to the global variable `msg`, and the handy property on it which can use here is called `sender`. 

Feel free to play around with `msg.sender` in a separate _spike project._ You should see that Solidity uses a special type `address` to refer to this user's unique identifier, which is also just their wallet address.

<br/>

Phew, ok so after all that theory we have decided that we want to have a mapping of addresses to uints, and I'll name the mapping `UserToNumber`.

```solidity
mapping (address -> uint256) UserToNumber;
```

Now, let's think about our functions. When we call "getStoredData" we want it to return the value within our mapping when we use `msg.sender` as the key.

Written in Solidity, it should look something like this:
```solidity
function getStoredData() public view returns (uint256) {
    return UserToNumber[msg.sender];
}
```

We can implement "getStoredData" with an extremely similar syntax. We use the "=" assignment operator to our function argument value into our mapping where the key is `msg.sender`.

```
function setStoredData(uint256 num) public {
    UserToNumber[msg.sender] = num;
}
```

Yep, that's it!

Now let's run our tests again and see what happens:

```
  Contract: OwnerStorage
    Initial deployment
      ✓ should assert true
    Functionality
      ✓ should initialize each user's value to 0 (70ms)
      ✓ should store a value for each user (170ms)


  3 passing (504ms)

```

Woohoo! We're all green! ✅ 🥳

Feel free to take a break and dance around, and then let's tackle the "getCount" function! 

<br/>



## Tests For GetCount

Ok, let's get right into it tdd style!

When we call "getCount" as the owner, we want to get the user's count. Let's write a test where we get the count of two different users who each have a different number assigned in the mapping. To keep things simple we'll just use the numbers 1 for the first user and 2 for the second user.

```javascript
it('should return the specified user\'s value when called by contract owner', async () => {
    // get subject
    const ownerStorage = await OwnerStorage.deployed();

    // Set the values for the first three users to 1, 2 respectively.
    await ownerStorage.setStoredData(1, { from: accounts[0] });
    await ownerStorage.setStoredData(2, { from: accounts[1] });

    // get the actual values now
    const user1storedData = await ownerStorage.getCount.call(accounts[0], { from: accounts[0] });
    const user2storedData = await ownerStorage.getCount.call(accounts[1], { from: accounts[0] });

    // verify we changed the subject
    assert.equal(user1storedData, 1, `user1storedData: ${user1storedData} was not 1!`);
    assert.equal(user2storedData, 2, `user1storedData: ${user2storedData} was not 2!`);
})
```

Notice how when we actually call "getCount" in the test above, we do it "from: accounts[0]". In other words, we are invoking the function call _as the user owner (since the first account is the owner by default)._

We can write a similar test from the perpective of a non-owner user trying to call getCount where we call the same getCount functions but "from: accounts[1]" (as a user who is _**not**_ the owner. 

This failure test is also bit tricky because we have to assert that it was rejected / reverted. First we should understand how Solidity and Truffle like to throw and handle these error conditions, and then we need to write a test which expects _that_ to happen...

<br/>



## Aside on Require


Remember that Solidity functions actually _cost money_ in terms of ETH gas to run. If the user enters bad inputs or something weird happens, we can "revert" the transaction, in which case nothing is written to the blockchain and the caller's gas is refunded.

With the `require` keyword we are able to say, "if this condition is not true, then make the function revert".

Here's how we might require that the input of a function is equal to the string "foobar":

```solidity
function (arg: string) public {
    
    require(arg == "foobar");
    
    // do more things down here, safely assuming the arg is "foobar". 

}
```

Now that we have an idea of what syntax tools we want to use, let's get back to just implementing the bare minimum in our getCount function in order to get _happy path_ test as the contract owner working.

<br/>

## Implenting GetCount 

Okay, so we have a test that it trying to invoke "call" on our getCount function, but since we haven't even defined that function our test is saying it's undefined. I guess that seems logical!

So, let's create this function with a totally empty body and see what our test says.

We'll also make it public so that it can be called from our tests and users:

```solidity
function getCount() public returns (uint256) {

}
```

The output when running our tests here is a bit obscure, but basically it's complaining that our function returns nothing when in our test we said it should return a uint.

```
  4 passing (944ms)
  1 failing

  1) Contract: OwnerStorage
       Getting values for a specified address
         should return the specified user's value when called by contract owner:
     Error: Error: [number-to-bn] while converting number ["0x627306090abaB3A6e1400e9345bC60c78a8BEf57"] to BN.js instance, error: invalid number value. Value must be an integer, hex string, BN or BigNumber instance. Note, decimals are not supported. Given value: "0x627306090abaB3A6e1400e9345bC60c78a8BEf57"
      at Context.<anonymous> (test/owner-storage.js:61:61)
      at processTicksAndRejections (internal/process/task_queues.js:95:5)
```


Okay, lets try to satify our test here by looking up our supplied argument in the mapping we have, and then returning that value.

```solidity
function getCount(address _address) public view returns (uint256) {
    return UserToNumber[_address];
}
```

Now when we run our tests... hey, they pass!

This is awesome, but we still have a bit left. Remember, we want this function to ONLY be callable from the owner. 

Currently the owner can call it, but everyone else can call it as well!

We'll start by writing a test that calls getCount as a non-owner user and expects it to revert...  

<br/>

## Asserting The Revert

In the final bullet of the code challenge they give us a hint- that there is a Truffle Assertion plugin that can test for revertions... nice!

let's first install it in our project:
```bash
npm i truffle-assertions
```


Okay, now we can go back to testing our OwnerStorage "getCount" function in the case where it should revert.

Note: don't forget to "await" the truffleAssert.reverts function call! 
(Otherwise, your tests may pass when they shouldn't which would be very bad!)

```javascript
const truffleAssert = require('truffle-assertions');

it('should reject when called by a user who is not the contract owner', async () => {
    // get subject
    const ownerStorage = await OwnerStorage.deployed();

    // a non-owner calling getCount for any user should revert
    await truffleAssert.reverts(ownerStorage.getCount.call({ from: accounts[1] }, [accounts[0]]));
    await truffleAssert.reverts(ownerStorage.getCount.call({ from: accounts[1] }, [accounts[1]]));
})
```


But I guess we'll implement the modifier anyway!


<br/>


## Adding An OwnerOnly Modifier



Remeber, we have the global variable `msg.sender` to get the caller's addres.

Unforunately, we don't have any global `owner` variable to get the owner's address.

Not to worry, though!

We can make an abstract contract that assigns an `_owner` variable to the value of `msg.sender` when the contract is first created (ie. in the constructor).

I'll name this abstract contract "Ownable".


Ownable.sol
```solidity
//SPDX-License-Identifier: MIT
pragma solidity >=0.5.0 <0.8.0;

abstract contract Ownable {
    
    address _owner;

    constructor() {
      _owner = msg.sender;
    }

}
```

Then, we can import this file in our OwnerStorage file and have our contract extend it / inherit from it with the `is` keyword.

Snippet from OwnerStorage.sol:
```solidity
//SPDX-License-Identifier: MIT
pragma solidity >=0.5.0 <0.8.0;

import "./Ownable.sol";

contract OwnerStorage is Ownable {
```


Ok, so now that we have "_owner" available to use our modifier is actually pretty compact and straightforward.

Let's name the modifier "onlyOwner" and have it require that the values of `owner` and `msg.sender` are equal:

Don't forget the `_;` when writing Solidity modifiers!

```solidity
modifier onlyOwner {
    require(msg.sender == _owner);
    _;
}
```


And finally, let's use this onlyOwner modifer on our getCount function.
```solidity
function getCount(address _address) public view onlyOwner returns (uint256) {
```

Now, when we run our tests we should see all beautifully green checkmarks, giving us the confidence that our code works correctly and the freedom to refactor without fear of unexpectedly breaking things. Yay!


```
  Contract: OwnerStorage
    Initial deployment
      ✓ should assert true
    Getting And Setting Values
      ✓ should initialize each user's value to 0 (76ms)
      ✓ should store a value for each user (203ms)
    Getting values for a specified address
      ✓ should return the specified user's value when called by contract owner (197ms)
      ✓ should reject when called by a user who is not the contract owner (446ms)


  5 passing (1s)
```

Huzzah, we've completed the code challenge. Our tests are all passing and asserting the right things, so I guess we're done!


See you next time for more TDD Solidity coding! Bye bye for now! 👋

<br/>

