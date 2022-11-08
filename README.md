
# Buckeye CTF 2022 Writeups

### Twin Prime RSA
The specs of this problem are pretty standard. Given an RSA modulus and a corresponding ciphertext, find the RSA key. We are given:
`n = 20533399299284046407152274475522745923283591903629216665466681244661861027880216166964852978814704027358924774069979198482663918558879261797088553574047636844159464121768608175714873124295229878522675023466237857225661926774702979798551750309684476976554834230347142759081215035149669103794924363457550850440361924025082209825719098354441551136155027595133340008342692528728873735431246211817473149248612211855694673577982306745037500773163685214470693140137016315200758901157509673924502424670615994172505880392905070519517106559166983348001234935249845356370668287645995124995860261320985775368962065090997084944099`

`c = 786123694350217613420313407294137121273953981175658824882888687283151735932871244753555819887540529041840742886520261787648142436608167319514110333719357956484673762064620994173170215240263058130922197851796707601800496856305685009993213962693756446220993902080712028435244942470308340720456376316275003977039668016451819131782632341820581015325003092492069871323355309000284063294110529153447327709512977864276348652515295180247259350909773087471373364843420431252702944732151752621175150127680750965262717903714333291284769504539327086686569274889570781333862369765692348049615663405291481875379224057249719713021`

`e = 65537`

However, we are also given a very useful hint: Given that R can be factored into two primes `p` and `q`, we know that these two are twin primes. Twin primes are a very special pair of odd primes such that the difference between the two are 2: the minimal distance between a pair of odd primes. These don't come up very often, but when they do, they have a lot of mathematical properties that don't make it look too good for our RSA here.

Suppose that $p, q$ are primes such that $p = q+2$. Then because
$n  = pq$, it follows that $n = q(q+2) = q^2 + 2q$. Therefore, we now have a univariate quadratic equation $q^2 + 2q - n = 0$. Consider the quadratic discriminant, $\Delta = 2^2 - 4*(-n)*1 = 4 + 4n$. By the Quadratic Formula, we know that the two roots to this equation are
$\frac{-2 \pm \sqrt{\Delta}}{2} = \frac{-2 \pm \sqrt{4+4n}}{2} = \frac{-2 \pm 2\sqrt{n+1}}{2} = -1 \pm \sqrt{n+1}$ and taking the absolute value of both solutions yields our two prime factors of $n$.

```py
# Twin Prime RSA
import math
n = 20533399299284046407152274475522745923283591903629216665466681244661861027880216166964852978814704027358924774069979198482663918558879261797088553574047636844159464121768608175714873124295229878522675023466237857225661926774702979798551750309684476976554834230347142759081215035149669103794924363457550850440361924025082209825719098354441551136155027595133340008342692528728873735431246211817473149248612211855694673577982306745037500773163685214470693140137016315200758901157509673924502424670615994172505880392905070519517106559166983348001234935249845356370668287645995124995860261320985775368962065090997084944099
c = 786123694350217613420313407294137121273953981175658824882888687283151735932871244753555819887540529041840742886520261787648142436608167319514110333719357956484673762064620994173170215240263058130922197851796707601800496856305685009993213962693756446220993902080712028435244942470308340720456376316275003977039668016451819131782632341820581015325003092492069871323355309000284063294110529153447327709512977864276348652515295180247259350909773087471373364843420431252702944732151752621175150127680750965262717903714333291284769504539327086686569274889570781333862369765692348049615663405291481875379224057249719713021
e = 65537

p = abs(sqrt(n+1)-1)
q = abs(-sqrt(n+1)-1)
d = pow(e, -1, (p-1)*(q-1))
m = str(hex(pow(c,d,n)))[2:]
flag = bytes.fromhex(m).decode()
print(f"Your flag is: {flag}")
```
<details>
  <summary>Flag:</summary>
 
 `buckeye{B3_TH3R3_OR_B3_SQU4R3__abcdefghijklmonpqrstuvwxyz__0123456789}`
 
</details>

### Quad Prime RSA

This RSA key challenge is a fair bit harder than the last one. Instead of simply having 2 twin primes making up our modulus, we are instead given 2 different moduli ciphertext pairs, $\langle n_1, c_1 \rangle, \langle n_2, c_2 \rangle$, all taken with the same public exponent $e = 65537$. In particular, $n_1$ is the product of some 1024-bit prime $q$, and some 500-bit prime, $p$ while $n_2$ is the product of some 1024 bit prime $r$ and some 500-bit prime $s$. It's important to note that $r$ and $q$ are twin primes, namely $r = q+2$. So therefore, $$n_1 = pq$$ $$n_2 = (q+2)s.$$ Because $r - q = 2$, we know that $2p + n_1 \equiv 0 \pmod r$. However, note that since $2p + n_1 = 2p+pq = p(q+2) = pr$, which obviously divides itself, it follows that $$2p + n_1 \equiv 0 \pmod {pr}$$ The polynomial $2x + n_1$ over the ring $\mathbb{Z}{\left[n_1n_2\right]}$ is a univariate polynomial (monic form being $x + n_1/2$) which has some small root, $p$ modulo $pr$. Therefore, by Coppersmith's Theorem, it suffices to extract a small root modulo $n_1n_2$, the result of which will be $p \mod {pr}$ where $pr$ is a divisor of $n_1n_2 = pqrs$. It follows that $pr \approx (n_1n_2)^{0.5}$ and therefore, the root size is $$p \approx (pr)^{0.5} \approx (n_1n_2)^{1/4} = (n_1n_2)^{\frac{0.5^2}{1}}$$ which lends us a direct application of the coppersmith method. We can use sage math's small_roots function with a beta value of 0.5 to achieve this, extracting $p$ and finding the flag.

```py
n_1 = 266809852588733960459210318535250490646048889879697803536547660295087424359820779393976863451605416209176605481092531427192244973818234584061601217275078124718647321303964372896579957241113145579972808278278954608305998030194591242728217565848616966569801983277471847623203839020048073235167290935033271661610383018423844098359553953309688771947405287750041234094613661142637202385185625562764531598181575409886288022595766239130646497218870729009410265665829
n_2 = 162770846172885672505993228924251587431051775841565579480252122266243384175644690129464185536426728823192871786769211412433986353757591946187394062238803937937524976383127543836820456373694506989663214797187169128841031021336535634504223477214378608536361140638630991101913240067113567904312920613401666068950970122803021942481265722772361891864873983041773234556100403992691699285653231918785862716655788924038111988473048448673976046224094362806858968008487
c_1 = 90243321527163164575722946503445690135626837887766380005026598963525611082629588259043528354383070032618085575636289795060005774441837004810039660583249401985643699988528916121171012387628009911281488352017086413266142218347595202655520785983898726521147649511514605526530453492704620682385035589372309167596680748613367540630010472990992841612002290955856795391675078590923226942740904916328445733366136324856838559878439853270981280663438572276140821766675
c_2 = 111865944388540159344684580970835443272640009631057414995719169861041593608923140554694111747472197286678983843168454212069104647887527000991524146682409315180715780457557700493081056739716146976966937495267984697028049475057119331806957301969226229338060723647914756122358633650004303172354762801649731430086958723739208772319851985827240696923727433786288252812973287292760047908273858438900952295134716468135711755633215412069818249559715918812691433192840

P.<x> = Zmod(n_1*n_2)[] #Polynomial Integer Ring over n_1n_2
f = 2*x + n_1
p = int(f.monic().small_roots(X=2^500, beta=0.5)[0]) #extract p
q = n_1//p
r = q+2
s = n_2//r
e = 65537
d_1 = pow(e, -1, (p - 1) * (q - 1))
d_2 = pow(e, -1, (r - 1) * (s - 1))
m_1 = str(hex(pow(c_1, d_1, n_1)))[2:]
m_2 = str(hex(pow(c_2, d_2, n_2)))[2:]
# print(m_1 == m_2): True
flag1 = bytes.fromhex(m_1).decode()
print(f"Your flag is: {flag1}")
```


<details>
  <summary>Flag:</summary>
 
 `buckeye{I_h0p3_y0u_us3D_c0nt1nu3d_fr4ct10Ns...th4nk5_d0R5A_f0r_th3_1nsp1r4t10n}`
 
</details>



An alternative is to analyze the bivariate polynomial $(2x + n_1)(n_2 - 2y) - n_1n_2$ over $\mathbb{Z}\left[n_1n_2\right]$, which has small roots $p, s$.
For this, we can use [this implentation of Coppersmith for bivariate polynomials](https://github.com/ubuntor/coppersmith-algorithm/blob/main/coppersmith.sage) and extract $p, s$. Note that upon analysis in the debug console, only $s$ is prime here (the approximation $p$ is only 1 off), but $s$ is all we need.

```py
from Crypto.Util.number import *
n = n_1 * n_2
X = Y = 2^500 # bounds on x0 and y0

P.<x,y> = PolynomialRing(ZZ)
pol = (2 * x + n1) * (n2 - 2 *y) - n1 * n2     # Should have a root at (x0,y0)

p,s = coron(pol, X, Y, k=2, debug=True)[0] #upon analysis, s is prime and p is just 1 more than the value from the coppersmith engine
if isPrime(p) or isPrime(s):
    print("Recovered")

r = n_2//s
q = r-2
p = n_1//q
d_1 = pow(e, -1, (p - 1) * (q - 1))
d_2 = pow(e, -1, (r - 1) * (s - 1))
m_1 = str(hex(pow(c_1, d_1, n_1)))[2:]
m_2 = str(hex(pow(c_2, d_2, n_2)))[2:]
# print(m_1 == m_2): True
flag1 = bytes.fromhex(m_1).decode()
print(f"Your flag is: {flag1}")
```
