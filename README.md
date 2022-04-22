# Write Up crewCTF 2022 kategori kriptografi

Saya Nafiz atau psudonym nya adalah ZafiN dari Cyber Security IPB, ingin memberikan writeup crewCTF 2022 kategori kriptografi. berikut writeup dari saya.

<!-- You can use the [editor on GitHub](https://github.com/MNafiz/ZafiN.github.io/edit/main/README.md) to maintain and preview the content for your website in Markdown files. -->

<!-- Whenever you commit to this repository, GitHub Pages will run [Jekyll](https://jekyllrb.com/) to rebuild the pages in your site, from the content in your Markdown files. -->

## Malleable Metal

### Diketahui bahwa n merupakan bilangan prima 8192 bit, sementara kunci publiknya adalah 3 yang menyebabkan msg1 dan msg2 dapat di recover menggunakan low exponent attack. lalu, karena r1 dan r2 lebih kecil dari 2 pangkat m, dimana m = 510. maka flag dapat di recover dengan membagi salah-satu dari msg1 dan msg2 dengan 2 pangkat m.
<!-- Markdown is a lightweight and easy-to-use syntax for styling your writing. It includes conventions for -->

```markdown
Diberikan source code enkripsi RSA dan hasil enkripsinya. Berikut source code nya.

`
from Crypto.PublicKey import RSA
from Crypto.Util.number import bytes_to_long
import random
import binascii
from secret import flag

e = 3
BITSIZE =  8192
key = RSA.generate(BITSIZE)
n = key.n
flag = bytes_to_long(flag)
m = floor(BITSIZE/(e*e)) - 400
assert (m < BITSIZE - len(bin(flag)[2:]))
r1 = random.randint(1,pow(2,m))
r2 = random.randint(r1,pow(2,m))
msg1 = pow(2,m)*flag + r1
msg2 = pow(2,m)*flag + r2
C1 = Integer(pow(msg1,e,n))
C2 = Integer(pow(msg2,e,n))
print(f'{n = }\n{C1 = }\n{C2 = }')
`

Berikut solvernya
`
>>> from Crypto.Util.number import *
>>> exec(open('out.txt').read())
>>> from gmpy2 import iroot
>>> msg1 = int(iroot(C1,3)[0])
>>> flag = msg1//pow(2,510)
>>> long_to_bytes(flag)
b"crew{l00ks_l1k3_y0u_h4v3_you_He4rd_0f_c0pp3rsm1th_sh0r+_p4d_4tt4ck_th4t_w45n't_d1ff1cult_w4s_it?}"
`
```

## Huge e

### Diketahui bahwa modulus merupakan bilangan prima sehingga mudah mendapatkan private key nya. akan tetapi, e bernilai sangat besar. hal tersebut dapat diatasi dengan menggunakan fermat little theorem dimana jika gcd(a,n) = 1, maka pow(a,b,n) = pow(a,b % phi(n),n). dimana phi(n) adalah banyaknya bilangan relatif prima dengan n yang lebih kecil dari n.  

```markdown
Diberikan source code enkripsi RSA dan hasil enkripsinya. Berikut source code nya.
`
from Crypto.Util.number import getPrime, bytes_to_long, inverse, isPrime
from secret import flag

m = bytes_to_long(flag)

def getSpecialPrime():
    a = 2
    for i in range(40):
        a*=getPrime(20)
    while True:
        b = getPrime(20)
        if isPrime(a*b+1):
            return a*b+1


p = getSpecialPrime()
e1 = getPrime(128)
e2 = getPrime(128)
e3 = getPrime(128)

e = pow(e1,pow(e2,e3))
c = pow(m,e,p)

assert pow(c,inverse(e,p-1),p) == m

print(f'p = {p}')
print(f'e1 = {e1}')
print(f'e2 = {e2}')
print(f'e3 = {e3}')
print(f'c = {c}')
`

Berikut solvernya
`
>>> from Crypto.Util.number import *
>>> from gmpy2 import next_prime
>>> exec(open('outt.txt').read())
>>> faktor = []
>>> tes = 1<<19
>>> while(len(faktor) <= 40):
...     tes = next_prime(tes)
...     if (p-1) % tes == 0:
...             faktor.append(int(tes))
... 
>>> phi = 1
>>> for i in faktor:
...     phi *= i-1
... 
>>> e2e3 = pow(e2,e3,phi)
>>> e = pow(e1,e2e3,p-1)
>>> d = pow(e,-1,p-1)
>>> long_to_bytes(pow(c,d,p))
b'crew{7hi5_1s_4_5ma11er_numb3r_7han_7h3_Gr4ham_numb3r}'
`
```

## ToyDL

### Diberikan banyak ciphertext hasil enkripsi RSA, kunci publiknya, serta puluhan hint yang diberikan soal (tetapi saya hanya menggunakan salah satu hintnya saja). saya menggunakan shor algorithm untuk mendapatkan faktor-faktor dari modulus sehingga mendapatkan kunci privat yang akan digunakan untuk mendapatkan flag

```markdown
Berikut hasil enkripsi, kunci publik, dan hint yang saya pilih.
`
== proof-of-work: disabled ==
n: 9099069576005010864322131238316022841221043338895736227456302636550336776171968946298044005765927235002236358603510713249831486899034262930368203212096032559091664507617383780759417104649503558521835589329751163691461155254201486010636703570864285313772976190442467858988008292898546327400223671343777884080302269

Here is RSA encrypted flag.(e=65537)
7721448675656271306770207905447278771344900690929609366254539633666634639656550740458154588923683190330091584419635454991419701119568903552077272516472473602367188377791329158090763546083264422552335660922148840678536264063681459356778292303287448582918945582522946194737497041408425657842265913159282583371732459

Let's see a property of numbers... I like small numbers.

hint:
pow(8, x, n) = 4
x=>3033023192001670288107377079438674280407014446298578742485434212183445592057322982099348001921975745000745452867836904416610495633011420976789401070698677517671018048439803949478138644614971957920195917451446560736395792884091775127542112411446073397768200019661315908739373825618842575924012884553576728676754666
`

Berikut solvernya
`
>>> from Crypto.Util.number import *
>>> n = 9099069576005010864322131238316022841221043338895736227456302636550336776171968946298044005765927235002236358603510713249831486899034262930368203212096032559091664507617383780759417104649503558521835589329751163691461155254201486010636703570864285313772976190442467858988008292898546327400223671343777884080302269
>>> e = 65537
>>> c = 7721448675656271306770207905447278771344900690929609366254539633666634639656550740458154588923683190330091584419635454991419701119568903552077272516472473602367188377791329158090763546083264422552335660922148840678536264063681459356778292303287448582918945582522946194737497041408425657842265913159282583371732459
>>> x = 3033023192001670288107377079438674280407014446298578742485434212183445592057322982099348001921975745000745452867836904416610495633011420976789401070698677517671018048439803949478138644614971957920195917451446560736395792884091775127542112411446073397768200019661315908739373825618842575924012884553576728676754666
>>> pow(8,x,n) == 4
True
>>> pow(2, 3*x - 2 , n) == 1
True
>>> pow(2, (3*x - 2)//2 , n)
1
>>> pow(2, (3*x - 2)//4 , n)
2698337488894070485449820579440827223270140789853821940822271010105316447908943787454463182157377497680260662990025884408464000205707444143110131638976574679615792812042805224374145172743364903073816924681290384064845042585097436824546832221018661399306348125482599884611934194579548874955690305522175254326356329
>>> GCD(pow(2, (3*x - 2)//4 , n) + 1,n)
2667409485887452272770831798706991010944832728754072378737171107257239406526140122282372492002898629230454157958918945658178559315566157078990098518311104787
>>> p = GCD(pow(2, (3*x - 2)//4 , n) + 1,n)
>>> q = n//p
>>> d = pow(e, -1 ,(p-1)*(q-1))
>>> long_to_bytes(pow(c,d,n))
b'@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@crew{d15cr373_l06_15_r3duc710n_f0r_f4c70r1n6}@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@'
`
```
<!-- Syntax highlighted code block

# Header 1
## Header 2
### Header 3

- Bulleted
- List

asdasdas

1. Numbered and
2. List

**Bold** and _Italic_ and `Code` text

[Link](url) and ![Image](src) -->

<!-- For more details see [Basic writing and formatting syntax](https://docs.github.com/en/github/writing-on-github/getting-started-with-writing-and-formatting-on-github/basic-writing-and-formatting-syntax).

### Jekyll Themes

Your Pages site will use the layout and styles from the Jekyll theme you have selected in your [repository settings](https://github.com/MNafiz/ZafiN.github.io/settings/pages). The name of this theme is saved in the Jekyll `_config.yml` configuration file.

### Support or Contact

Having trouble with Pages? Check out our [documentation](https://docs.github.com/categories/github-pages-basics/) or [contact support](https://support.github.com/contact) and we’ll help you sort it out. -->
