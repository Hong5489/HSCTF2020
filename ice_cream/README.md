# Ice Cream Bytes
> Introducing the new Ice Cream Bytes machine! Here’s a free trial: [IceCreamBytes.java](IceCreamBytes.java) Oh, and make sure to read the user manual: [IceCreamManual.txt](IceCreamManual.txt)

> Author: wooshi

Compile the `IceCreamBytes.java` using `javac` command, then run it with `java` command
```bash
javac IceCreamBytes.java
java IceCreamBytes
```
Then it prompt me for input the password:
```
java IceCreamBytes
Enter the password to the ice cream machine: password
Exception in thread "main" java.lang.ArrayIndexOutOfBoundsException: Index 2 out of bounds for length 2
	at IceCreamBytes.main(IceCreamBytes.java:19)
```
Looks like we got an error!

Check out the source code:
```java
byte[] manualBytes = Files.readAllBytes(path);

String input = userInput.substring("flag{".length(), userInput.length()-1);


byte[] loadedBytes = toppings(chocolateShuffle(vanillaShuffle(strawberryShuffle(input.getBytes()))));

byte[] correctBytes = fillMachine(manualBytes);
for (int i = 0; i < correctBytes.length; i++) {
    if (loadedBytes[i] != correctBytes[i]) {
        correctPassword  = false;
    }
}
```
`correctBytes` is come from `fillMachine` function:
```java
public static byte[] fillMachine(byte[] inputIceCream) {
    byte[] outputIceCream = new byte[34];
    int[] intGredients = {27, 120, 79, 80, 147, 
        154, 97, 8, 13, 46, 31, 54, 15, 112, 3, 
        464, 116, 58, 87, 120, 139, 75, 6, 182, 
        9, 153, 53, 7, 42, 23, 24, 159, 41, 110};
    for (int i = 0; i < outputIceCream.length; i++) {
        outputIceCream[i] = inputIceCream[intGredients[i]];
    }
    return outputIceCream;
}
```
Looks like our input should be in **length of 34 and with flag format**

Let's test it out:
```
java IceCreamBytes
Enter the password to the ice cream machine: flag{aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa}
Uhhh that's not right.
```
It ran without error!

Let's analyze the source code abit more:
```java
// Get our input and go though 4 shuffle functions
byte[] loadedBytes = toppings(chocolateShuffle(vanillaShuffle(strawberryShuffle(input.getBytes()))));
boolean correctPassword = true;

// Then it compare it with the bytes generated from fillMachine
byte[] correctBytes = fillMachine(manualBytes);
for (int i = 0; i < correctBytes.length; i++) {
    if (loadedBytes[i] != correctBytes[i]) {
        correctPassword  = false;
    }
}

// If all match then it is the correct flag!
if (correctPassword) {
    System.out.println("That's right! Enjoy your ice cream!");
} else {
    System.out.println("Uhhh that's not right.");
}
```
So the whole process basically is like this:
```
input -> strawberryShuffle -> vanillaShuffle -> chocolateShuffle -> toppings = correctBytes
```
We can just reverse the process with the given correctBytes generated by fillMachine then we will able to get input (flag):
```
correctBytes -> toppings -> chocolateShuffle -> vanillaShuffle -> strawberryShuffle = input
```
In java code will look like this:
```java
byte[] correctBytes = fillMachine(manualBytes);
strawberryShuffle(vanillaShuffle(chocolateShuffle(toppings(correctBytes))));
```
But before we reverse it, we also need reverse the function code.

Like plus become minus, minus become plus etc.

1. vanillaShuffle
`+` to `-`, and `-` to `+`
```java
if (i % 2 == 0) {
    outputIceCream[i] = (byte)(inputIceCream[i] - 1);
} else {
    outputIceCream[i] = (byte)(inputIceCream[i] + 1);
}
```

2. chocolateShuffle
Exchange the index from left to right, right to left

| Before  | After |
| ------------- | ------------- |
| outputIceCream[i] = inputIceCream[i - 2];  | outputIceCream[i - 2] = inputIceCream[i];  |
|  outputIceCream[i] = inputIceCream[inputIceCream.length - 2];  |  outputIceCream[inputIceCream.length - 2] = inputIceCream[i];  |
| outputIceCream[i] = inputIceCream[i + 2];| outputIceCream[i + 2] = inputIceCream[i];|
|outputIceCream[i] = inputIceCream[1];|outputIceCream[1] = inputIceCream[i];|

3. toppings
Just `+` to `-`
```java
outputIceCream[i] = (byte)(inputIceCream[i] + toppings[i]);
```
Then we can edit the `main` function code to print out the flag for us!
```java
public static void main(String[] args) throws IOException {
    Path path = Paths.get("IceCreamManual.txt");
    byte[] manualBytes = Files.readAllBytes(path);

    byte[] correctBytes = fillMachine(manualBytes);
    System.out.println(new String(strawberryShuffle(vanillaShuffle(chocolateShuffle(toppings(correctBytes))))));
}
```
I saved it as [Decode.java](Decode.java)

Then compile and run it:
```
javac Decode.java
java Decode
ic3_cr34m_byt3s_4r3_4m4z1n9_tr34ts
```
Yay! That's the flag!!

Confirmed with the original program:
```
Enter the password to the ice cream machine: flag{ic3_cr34m_byt3s_4r3_4m4z1n9_tr34ts}
That's right! Enjoy your ice cream!
```

## Flag
```
flag{ic3_cr34m_byt3s_4r3_4m4z1n9_tr34ts}
```