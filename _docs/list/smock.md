# Smock readme

# Getting Started

## Installation

.. tabs::

.. group-tab:: yarn

    .. code-block:: text

      yarn add --dev @defi-wonderland/smock

.. group-tab:: npm

    .. code-block:: text

      npm install --save-dev @defi-wonderland/smock

## Required Config for Mocks

`Mocks <./mocks.html>`\_ allow you to manipulate any variable inside of a smart contract.
If you'd like to use mocks, you **must** update your :code:`hardhat.config.<js/ts>` file to include the following:

.. tabs::

.. group-tab:: JavaScript

    .. code-block:: javascript

      // hardhat.config.js

      ... // your plugin imports and whatnot go here

      module.exports = {
        ... // your other hardhat settings go here
        solidity: {
          ... // your other Solidity settings go here
          compilers: [
            ...// compiler options
            settings: {
              outputSelection: {
                "*": {
                  "*": ["storageLayout"]
                }
              }
            }
          ]
        }
      }

.. group-tab:: TypeScript

    .. code-block:: typescript

      // hardhat.config.js

      ... // your plugin imports and whatnot go here

      const config = {
        ... // your other hardhat settings go here
        solidity: {
          ... // your other Solidity settings go here
          compilers: [
            ...// compiler options
            settings: {
              outputSelection: {
                "*": {
                  "*": ["storageLayout"]
                }
              }
            }
          ]
        }
      }

      export default config

# Fakes

## What are fakes?

Fakes are JavaScript objects that emulate the interface of a given Solidity contract.
You can use fakes to customize the behavior of any public method or variable that a smart contract exposes.

## When should I use a fake?

Fakes are a powerful tool when you want to test how a smart contract will interact with other contracts.
Instead of initalizing a full-fledged smart contract to interact with, you can simply create a fake that can provide pre-programmed responses.

Fakes are especially useful when the contracts that you need to interact with are relatively complex.
For example, imagine that you're testing a contract that needs to interact with another (very stateful) contract.
Without :code:`smock`, you'll probably have to:

1. Deploy the contract you need to interact with.
2. Perform a series of transactions to get the contract into the relevant state.
3. Run the test.
4. Do this all over again for each test.

This is annoying, slow, and brittle.
You might have to update a bunch of tests if the behavior of the other contract ever changes.
Developers usually end up using tricks like state snapshots and complex test fixtures to get around this problem.
Instead, you can use :code:`smock`:

1. Create a :code:`fake`.
2. Make your :code:`fake` return the value you want it to return.
3. Run the test.

## Using fakes

Initialization

---

Initialize with a contract name
###############################

.. code-block:: typescript

const myFake = await smock.fake('MyContract');

Initialize with a contract ABI
##############################

.. code-block:: typescript

const myFake = await smock.fake([ { ... } ]);

Initialize with a contract factory
##################################

.. code-block:: typescript

const myContractFactory = await hre.ethers.getContractFactory('MyContract');
const myFake = await smock.fake(myContractFactory);

Initialize with a contract instance
###################################

.. code-block:: typescript

const myContractFactory = await hre.ethers.getContractFactory('MyContract');
const myContract = await myContractFactory.deploy();
const myFake = await smock.fake(myContract);

Take full advantage of typescript and typechain
###############################################

.. code-block:: typescript

const myFake = await smock.fake<MyContract>('MyContract');

Options
#######

.. code-block:: typescript

await smock.fake('MyContract', { ... }); // how to use

// options
{
address?: string; // initialize fake at a specific address
provider?: Provider; // initialize fake with a custom provider
}

Signing transactions

---

Every fake comes with a :code:`wallet` property in order to make easy to sign transactions

.. code-block:: typescript

myContract.connect(myFake.wallet).doSomething();

Making a function return

---

Returning with the default value
################################

.. code-block:: typescript

myFake.myFunction.returns();

Returning a fixed value
#######################

.. code-block:: typescript

myFake.myFunction.returns(42);

Returning a struct
##################

.. code-block:: typescript

myFake.getStruct.returns({
valueA: 1234,
valueB: false,
});

Returning an array
##################

.. code-block:: typescript

myFake.myFunctionArray.returns([1, 2, 3]);

Returning a dynamic value
#########################

.. code-block:: typescript

myFake.myFunction.returns(() => {
if (Math.random() < 0.5) {
return 0;
} else {
return 1;
}
});

Returning a value based on arguments
####################################

.. code-block:: typescript

myFake.myFunction.whenCalledWith(123).returns(456);

await myFake.myFunction(123); // returns 456

Returning a value with custom logic
###################################

.. code-block:: typescript

myFake.getDynamicInput.returns(arg1 => arg1 \* 10);

await myFake.getDynamicInput(123); // returns 1230

Returning at a specific call count
##################################

.. code-block:: typescript

myFake.myFunction.returnsAtCall(0, 5678);
myFake.myFunction.returnsAtCall(1, 1234);

await myFake.myFunction(); // returns 5678
await myFake.myFunction(); // returns 1234

Making a function revert

---

Reverting with no data
######################

.. code-block:: typescript

myFake.myFunction.reverts();

Reverting with a string message
###############################

.. code-block:: typescript

myFake.myFunction.reverts('Something went wrong');

Reverting with bytes data
#########################

.. code-block:: typescript

myFake.myFunction.reverts('0x12341234');

Reverting at a specific call count
##################################

.. code-block:: typescript

myFake.myFunction.returns(1234);
myFake.myFunction.revertsAtCall(1, 'Something went wrong');

await myFake.myFunction(); // returns 1234
await myFake.myFunction(); // reverts with 'Something went wrong'
await myFake.myFunction(); // returns 1234

Reverting based on arguments
############################

.. code-block:: typescript

myFake.myFunction.returns(1);
myFake.myFunction.whenCalledWith(123).reverts('Something went wrong');

await myFake.myFunction(); // returns 1
await myFake.myFunction(123); // reverts with 'Something went wrong'

Resetting function behavior

---

Resetting a function to original behavior
#########################################

.. code-block:: typescript

myFake.myFunction().reverts();

await myFake.myFunction(); // reverts

myFake.myFunction.reset(); // resets behavior for all inputs of the function

await myFake.myFunction(); // returns 0

Asserting call count

---

Any number of calls
###################

.. code-block:: typescript

expect(myFake.myFunction).to.have.been.called;

Called once
###########

.. code-block:: typescript

expect(myFake.myFunction).to.have.been.calledOnce;

Called twice
############

.. code-block:: typescript

expect(myFake.myFunction).to.have.been.calledTwice;

Called three times
##################

.. code-block:: typescript

expect(myFake.myFunction).to.have.been.calledThrice;

Called N times
##############

.. code-block:: typescript

expect(myFake.myFunction).to.have.callCount(123);

Asserting call arguments or value

---

Called with specific arguments
##############################

.. code-block:: typescript

expect(myFake.myFunction).to.have.been.calledWith(123, true, 'abcd');

Called with struct arguments
############################

.. code-block:: typescript

expect(myFake.myFunction).to.have.been.calledWith({
myData: [1, 2, 3, 4],
myNestedStruct: {
otherValue: 5678
}
});

Called at a specific call index with arguments
##############################################

.. code-block:: typescript

expect(myFake.myFunction.atCall(2)).to.have.been.calledWith(1234, false);

Called once with specific arguments
###################################

.. code-block:: typescript

expect(myFake.myFunction).to.have.been.calledOnceWith(1234, false);

Called with an specific call value
###################################

.. code-block:: typescript

expect(myFake.myFunction).to.have.been.calledWithValue(1234);

Asserting call order

---

Called before other function
############################

.. code-block:: typescript

expect(myFake.myFunction).to.have.been.calledBefore(myFake.myOtherFunction);

Called after other function
###########################

.. code-block:: typescript

expect(myFake.myFunction).to.have.been.calledAfter(myFake.myOtherFunction);

Called immediately before other function
########################################

.. code-block:: typescript

expect(myFake.myFunction).to.have.been.calledImmediatelyBefore(myFake.myOtherFunction);

Called immediately after other function
#######################################

.. code-block:: typescript

expect(myFake.myFunction).to.have.been.calledImmediatelyAfter(myFake.myOtherFunction);

Querying call arguments

---

Getting arguments at a specific call index
##########################################

.. code-block:: typescript

expect(myFake.myFunction.getCall(0).args[0]).to.be.gt(50);

Getting call value at a specific call index
##########################################

.. code-block:: typescript

expect(myFake.myFunction.getCall(0).value).to.eq(1);

Manipulating fallback functions

---

Modifying the :code:`fallback` function
#######################################

.. code-block:: typescript

myFake.fallback.returns();

Modifying the :code:`receive` function
######################################

.. code-block:: typescript

myFake.receive.returns();

Delegated calls

---

Calls to a contract function via delegated calls do behave the same as a regular call, so you can enforce a return value, assert the calls details, etc...
In addition, you also have custom assertions for delegated calls.

Assert delegated caller
#######################

.. code-block:: typescript

expect(myFake.myFunction).to.be.delegatedFrom(myProxy.address);

# Mocks

## What are mocks?

Mocks are extensions to smart contracts that have all of the functionality of a `fake <./fakes.html>`\_ with some extra goodies.
Behind every mock is a real smart contract (with actual Solidity code!) of your choosing.
You can **modify the behavior of functions like a fake**, or you can leave the functions alone and calls will pass through to your actual contract code.
And, with a little bit of smock magic, you can even **modify the value of variables within your contract**! 🥳

## When should I use a mock?

Generally speaking, mocks are more advanced versions of `fakes <./fakes.html>`\_.
Mocks are most effectively used when you need _some_ behavior of a real smart contract but still want the ability to modify things on the fly.

One powerful feature of a mock is that you can modify the value of variables within the smart contract.
You could, for example, use this feature to test the behavior of a function that changes behavior depending on the value of a variable.

## Using mocks

Initialization

---

Initialize with a contract name
###############################

.. code-block:: typescript

const myContractFactory = await smock.mock('MyContract');
const myContract = await myContractFactory.deploy(...);

Take full advantage of typescript and typechain
###############################################

.. code-block:: typescript

await smock.mock<MyContract\_\_factory>('MyContract');

Options
#######

.. code-block:: typescript

await smock.mock('MyContract', { ... }); // how to use

// options
{
provider?: Provider; // initialize mock with a custom provider
}

Using features of fakes

---

Mocks can use any feature available to fakes.
See the documentation of `fakes <./fakes.html>`\_ for more information.

Call through

---

Calls go through to contract by default
#######################################

.. code-block:: typescript

await myMock.add(10);
await myMock.count(); // returns 10

myMock.count.returns(1);
await myMock.count(); // returns 1

Manipulating variables

---

.. warning::
This is an experimental feature and it is subject to API changes in the near future

Setting the value of a variable
###############################

.. code-block:: typescript

await myMock.setVariable('myVariableName', 1234);

Setting the value of a struct
#############################

.. code-block:: typescript

await myMock.setVariable('myStruct', {
valueA: 1234,
valueB: true,
});

Setting the value of a mapping (won't affect other keys)
########################################################

.. code-block:: typescript

await myMock.setVariable('myMapping', {
myKey: 1234
});

Setting the value of a nested mapping
#####################################

.. code-block:: typescript

await myMock.setVariable('myMapping', {
myChildMapping: {
myKey: 1234
}
});

Setting the value of multiple variables
#######################################

.. code-block:: typescript

await myMock.setVariables({
myVariableName1: 123,
myVariableName2: true,
myStruct: {
valueA: 1234,
valueB: false,
},
myMapping: {
[myKey]: 1234
}
})

Getting the value of an internal variable

---

.. warning::
This is an experimental feature and it does not support multidimensional or packed arrays

.. code-block:: typescript

const myUint256 = await myMock.getVariable('myUint256VariableName');

Getting the value of an internal mapping given the value's key
#######################################

.. code-block:: typescript

const myMappingValue = await myMock.getVariable('myMappingVariableName', [mappingKey]);

Getting the value of an internal nested mapping given the value's keys
#######################################

.. code-block:: typescript

const myMappingValue = await myMock.getVariable('myMappingVariableName', [mappingKeyA, mappingKeyB]);
