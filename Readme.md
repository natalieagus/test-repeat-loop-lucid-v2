## `repeat` loop investigation 

**AlchitryLabsV2 v2.0.9** preview was used. 

This project contains a simple implementation of N bit Ripple-carry-adder (`rca.luc`)


```
// A ripple-carry-adder combinational logic unit
module rca #(
    SIZE = 2 : SIZE > 1
)(
    input a[SIZE],
    input b[SIZE],
    input cin,
    output cout,
    output s[SIZE],
) {

    fa fa[SIZE];
    
    always {
        fa.a = a;
        fa.b = b;
        fa.cin[0] = cin;
        
        // plain connection works 
         fa.cin[SIZE-1:1] = fa.cout[SIZE-2:0]
        
        cout = fa.cout[SIZE-1];
        s = fa.s;

    }
}
```

The `fa` unit is just a simple 1-bit full adder module:
```
// A full adder combinational logic unit 
module fa (
    output s,
    output cout,
    input a,
    input b,
    input cin
) {
    always {
        s = a ^ b ^ cin;
        cout = (a & b) + (a & cin) + (b & cin);
    }
}
```

We try to replace each `fa`'s `cin, cout` connection with a `repeat` loop, and came up with 3 versions that should (logically) works the same. 

```
        // version 1
         repeat(i, SIZE-1, 0){
            fa.cin[i+1] = fa.cout[i];
        }

        // version 2
        repeat(i, SIZE-1, SIZE-1, -1){
            fa.cin[i] = fa.cout[i-1];
        }

       // version 3
        repeat(i, SIZE-1, 1){
            fa.cin[i] = fa.cout[i-1];
        }
```

Here's the observed behavior:
- **version 1**:  works in simulation correctly for all values of `SIZE` > 2. This behaves the same as plain connection above. 
- **version 2**: if I set `SIZE` as 3, the following error shows. There's no error with **other** `SIZE` value e.g: `2,4,5,6...`. However, simulation shows the **wrong** output,  e.g: 1+1 does not give 2, but gives 10. 
<img width="778" alt="image" src="https://github.com/alchitry/Alchitry-Labs-V2/assets/5848920/4a0f7059-b851-46b7-86f9-2da8a59fe8eb">
- **version 3**:  same behavior as **version 2**

Restarted Alchitry-Labs-V2 (2.0.9) (just in case there's simulation cache), when testing each version.
