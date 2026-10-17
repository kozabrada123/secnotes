
a^n where n [1, p-1) mod p doesn't always generate the entire set [1, p-1) !

If a is a residue, then a^n is naturally also a residue. It's worth checking if it is, because then it doesn't generate the whole set

----

a mod n = ab mod nb

----

if you have 
    p = (a + 2b)^c mod N
and 
    q = (2a + b)^d mod N 

where N = ab,

you can say p = a^c mod b and q = (2a)^d mod b,
rearrange everything, get

a^c = p mod b
2^d * a^d = q mod b

a ^ c = p mod b
a ^ d = q * 2^-d mod b

a ^ cd = p^d mod b
a ^ cd = q^c * 2^-cd mod b

THEN

p^d - (q^c * 2^-cd) = 0 mod b

(because they are equal!!)

p^d * 2^cd - q^c = 0 mod b

let A = p^d * 2^cd - q^c

A = jb

N = ab

gcd(A, N) = gcd(jb, ab) = b
