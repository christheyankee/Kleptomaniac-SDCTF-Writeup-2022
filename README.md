# Kleptomaniac
## Description
An internationally wanted league of bitcoin thieves have finally been tracked down to their internet lair! They are hiding behind a secure portal that requires guessing seemingly random numbers. After apprehending one of the thieves offline, the authorities were able to get a copy of the source code as well as an apparently secret value. Can you help us crack the case? We have no idea how to use this “backdoor number”...   
"Secret Value"   
```5b8c0adce49783789b6995ac0ec3ae87d6005897f0f2ddf47e2acd7b1abd```  
Connect via   
```nc lair.sdc.tf 1337```   
Source code   
```main.py```   
## Solve
Obviosuly we cannot predict the future. The first thing to note is that we can query the oracle an infinite amount of times. 
The goal here then is probably to leak information about the curve until we get to a point where we can predict the value.
So let's actually look at some of what the oracle is doing.
### The Nuts and Bolts
```
state = mulECPointScalar(NIST_256_CURVE, P, state)[0]
randval = mulECPointScalar(NIST_256_CURVE, Q, state)[0] & 0x00ffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff
```
So it is using the old state to generate the random value that we need to guess. A crucial thing to notice here is that the state
is only randomized on the first iteration. So what does the "secret value" do? After some poking around it becomes clear that
```secret_value * Q = P```. That's useful to know. Notice further that ```randval = Q * P * old_state```. Notice further that the next new
state is just ```P * P * old_state```. Then we know that ```secret_value * backdoor = new_state```. Thus we can predict the new state. But there's
a problem. remember that long hex value above? Its obfuscating one of the bytes. Thankfully there are only 2^8 values we need to try.
We create a table with all of the possible ```new_state * Q``` values and then query the oracle again. It will generate one of the values
in our table and we will know what the correct value for new_state is. Then we can predict the next random value, apply the obfuscation and get the flag.
### The Flag
```
sdctf{W0W_aR3_y0u_Th3_NSA}
```