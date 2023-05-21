Mass Assignment vulnerabilities are present when an attacker is able to overwrite object properties that they should not be able to.

Conditions:

(i) An API must have requests that accept user input, these requests must be able to alter values not available to the user

(ii) the API must be missing security controls that would otherwise prevent the user input from altering data objects.

### Finding Mass Assignment Vulnerabilities

(i) One of the ways that you can discover mass assignment vulnerabilities by finding interesting parameters in API documentation and then adding those parameters to requests. => Look for parameters involved in user account properties, critical functions, and administrative actions.

(ii) Additionally, make sure to use the API as it was designed so that you can study the parameters that are used by the API provider.

(iii) Doing this will help you understand the names and spelling conventions of the parameters that your target uses. If you find parameters used in some requests, you may be able to leverage those in your mass assignment attacks in other requests.

(iv) You can also test for mass assignment blind by fuzzing parameter values within requests. Mass assignment attacks like this will be necessary when your target API does not have documentation available.

(v)  Essentially, you will need to capture requests that accept user input and use tools to brute force potential parameters.

(vi)  I recommend starting out your search for mass assignment vulnerabilities by testing your target's account registration process if there is one.

### The challenge with mass assignment attacks is that there is very little consistency in the parameters used between API providers. That being said, if the API provider has some method for, say, designating accounts as administrators, they may also have some convention for creating or updating variables to make a user an administrator. Fuzzing can speed up your search for mass assignment vulnerabilities, but unless you understand your target’s variables, this technique can be a shot in the dark.



## Hunting for Mass Assignment

 mass assignment is all about binding user input to data objects. So, when you analyze a collection that you are targeting you will need to find requests that:

-   Accept user input
-   Have the potential to modify objects

## Fuzzing for MASS Assignment with Param Miner.
Using burpsuite's param-miner we can fuzz Parameters to get MASS Assignment Vector

