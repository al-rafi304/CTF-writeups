# Paul's Final Tune-Up
The challenge provided a link to the website and 3 source files **(server.js, package.json, public.pem)**.  
As this is a continuation of a JWT cracking challenge, the main exploit for this had to be related to JWT as well.

## Source Code Analysis
![image](https://github.com/user-attachments/assets/24755e83-72c8-4302-8b12-10e79dec8452)

It seems the `/validate` endpoint returns the flag based on two conditions: *the JWT token must be valid, and the `isUserRegistered` parameter must be true inside the token.*
If the token is invalid, the server will send an errorResponse and set the JWT cookie. However, if the token is valid but the isUserRegistered is set to false, it will return a successResponse.
Also, the algorithm used for signing the token is `RS256` which requires a public key and private key. We are only given the public key in the *public.pem* file.

## Identifying Vulnerability
![image](https://github.com/user-attachments/assets/e7816325-b16d-4468-9ba5-da53278428ba)

Now in the *package.json* file, we can see it's using `jsonwebtoken v4.0.0` which looked pretty old and older versions often consist of security vulnerabilities.
So a quick google search later I found a [CVE-2022-23539](https://nvd.nist.gov/vuln/detail/CVE-2022-23539) vulnerability for that package. 

According to my understanding, we can bypass the token verification by using the `HS256` algorithm instead of `RS256` because a poorly written `verify()` function.
Also the `HS256` only needs a single secret key so we can just provide the public key as a secret key to sign the token and it will still be validated.

## Exploit
![image](https://github.com/user-attachments/assets/822fdb7c-bc19-408c-9460-c1cf97ef3725)

I used the `JOSEPH` and `JSON Web Tokens` extensions in burp for forging the token. Using the above settings in `JOSEPH`, I generated a `HS256` JWT from the `RS256` JWT cookie the server set and sent a request to the endpoint to test my theory.

![image](https://github.com/user-attachments/assets/155d2d92-88d8-45ea-b560-421d4115cee5)

As you can see, the server sent a success response even though I signed the token using `HS256` algorithm so the theory was correct. But it didn't send the flag because we need to set `isUserRegistered` to **true**.
So I used the `JSON Web Tokens` tool to change the parameter value of the generated `HS256` JWT and recalculate the signature using the given public key. Sending a request with the forged JWT revealed the flag.

![image](https://github.com/user-attachments/assets/98357732-01c0-4e74-8878-63252292d79e)

**Note: This is how I did it in the competition last minute. So maybe it could've been done more easily or efficiently**