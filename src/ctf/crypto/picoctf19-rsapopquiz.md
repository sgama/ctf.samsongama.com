# PicoCTF19 RSA Pop Quiz

## Challenge

Class, take your seats! It's PRIME-time for a quiz... `nc 2019shell1.picoctf.com 2611`

### Hints

[RSA info](https://simple.wikipedia.org/wiki/RSA_algorithm)

## Solution

```
$ nc 2019shell1.picoctf.com 2611
Good morning class! It's me Ms. Adleman-Shamir-Rivest
Today we will be taking a pop quiz, so I hope you studied. Cramming just will not do!
You will need to tell me if each example is possible, given your extensive crypto knowledge.
Inputs and outputs are in decimal. No hex here!
#### NEW PROBLEM ####
q : 60413
p : 76753
##### PRODUCE THE FOLLOWING ####
n
```

We know `n=p*q`

```
#### TIME TO SHOW ME WHAT YOU GOT! ###
n: 4636878989
Outstanding move!!!


#### NEW PROBLEM ####
p : 54269
n : 5051846941
##### PRODUCE THE FOLLOWING ####
q
```

We know `n = q/n`

```
#### TIME TO SHOW ME WHAT YOU GOT! ###
q: 93089
Outstanding move!!!


#### NEW PROBLEM ####
e : 3
n : 12738162802910546503821920886905393316386362759567480839428456525224226445173031635306683726182522494910808518920409019414034814409330094245825749680913204566832337704700165993198897029795786969124232138869784626202501366135975223827287812326250577148625360887698930625504334325804587329905617936581116392784684334664204309771430814449606147221349888320403451637882447709796221706470239625292297988766493746209684880843111138170600039888112404411310974758532603998608057008811836384597579147244737606088756299939654265086899096359070667266167754944587948695842171915048619846282873769413489072243477764350071787327913
##### PRODUCE THE FOLLOWING ####
q
p
```
We know `toitent(n)=(p-1)(q-1)` but we don't have the toitent.

```
IS THIS POSSIBLE and FEASIBLE? (Y/N):N
Outstanding move!!!


#### NEW PROBLEM ####
q : 66347
p : 12611
##### PRODUCE THE FOLLOWING ####
totient(n)
```

We know: `toitent(n)=(p-1)(q-1)`

```
IS THIS POSSIBLE and FEASIBLE? (Y/N):Y
#### TIME TO SHOW ME WHAT YOU GOT! ###
totient(n): ^V836623060
Outstanding move!!!


#### NEW PROBLEM ####
plaintext : 6357294171489311547190987615544575133581967886499484091352661406414044440475205342882841236357665973431462491355089413710392273380203038793241564304774271529108729717
e : 3
n : 29129463609326322559521123136222078780585451208149138547799121083622333250646678767769126248182207478527881025116332742616201890576280859777513414460842754045651093593251726785499360828237897586278068419875517543013545369871704159718105354690802726645710699029936754265654381929650494383622583174075805797766685192325859982797796060391271817578087472948205626257717479858369754502615173773514087437504532994142632207906501079835037052797306690891600559321673928943158514646572885986881016569647357891598545880304236145548059520898133142087545369179876065657214225826997676844000054327141666320553082128424707948750331
##### PRODUCE THE FOLLOWING ####
ciphertext
```

We know: `c = plaintext^e mod n`

```


#### NEW PROBLEM ####
ciphertext : 107524013451079348539944510756143604203925717262185033799328445011792760545528944993719783392542163428637172323512252624567111110666168664743115203791510985709942366609626436995887781674651272233566303814979677507101168587739375699009734588985482369702634499544891509228440194615376339573685285125730286623323
e : 3
n : 27566996291508213932419371385141522859343226560050921196294761870500846140132385080994630946107675330189606021165260590147068785820203600882092467797813519434652632126061353583124063944373336654246386074125394368479677295167494332556053947231141336142392086767742035970752738056297057898704112912616565299451359791548536846025854378347423520104947907334451056339439706623069503088916316369813499705073573777577169392401411708920615574908593784282546154486446779246790294398198854547069593987224578333683144886242572837465834139561122101527973799583927411936200068176539747586449939559180772690007261562703222558103359
##### PRODUCE THE FOLLOWING ####
plaintext
```

We don't know `p` and `q`

```
IS THIS POSSIBLE and FEASIBLE? (Y/N):N
Outstanding move!!!


#### NEW PROBLEM ####
q : 92092076805892533739724722602668675840671093008520241548191914215399824020372076186460768206814914423802230398410980218741906960527104568970225804374404612617736579286959865287226538692911376507934256844456333236362669879347073756238894784951597211105734179388300051579994253565459304743059533646753003894559
p : 97846775312392801037224396977012615848433199640105786119757047098757998273009741128821931277074555731813289423891389911801250326299324018557072727051765547115514791337578758859803890173153277252326496062476389498019821358465433398338364421624871010292162533041884897182597065662521825095949253625730631876637
e : 65537
##### PRODUCE THE FOLLOWING ####
d
```

We know: `d=e^-1 mod ((p-1)(q-1))`

```python
from Crypto.Util.number import *

q = ''
p = ''
e = ''
print(inverse(e,((p-1)(q-1))))
```

Easy.

```
IS THIS POSSIBLE and FEASIBLE? (Y/N):Y
#### TIME TO SHOW ME WHAT YOU GOT! ###
d: 1405046269503207469140791548403639533127416416214210694972085079171787580463776820425965898174272870486015739516125786182821637006600742140682552321645503743280670839819078749092730110549881891271317396450158021688253989767145578723458252769465545504142139663476747479225923933192421405464414574786272963741656223941750084051228611576708609346787101088759062724389874160693008783334605903142528824559223515203978707969795087506678894006628296743079886244349469131831225757926844843554897638786146036869572653204735650843186722732736888918789379054050122205253165705085538743651258400390580971043144644984654914856729
Outstanding move!!!


#### NEW PROBLEM ####
p : 153143042272527868798412612417204434156935146874282990942386694020462861918068684561281763577034706600608387699148071015194725533394126069826857182428660427818277378724977554365910231524827258160904493774748749088477328204812171935987088715261127321911849092207070653272176072509933245978935455542420691737433
ciphertext : 4699954403535877728943212516495239996093493409461427795061606820019520385578403561120385764629211115765041521697969103538878070126128059106090044437598460283768854171495071441758538307495380993096127617485853022154997313813963653770523746165616397996160676397490439829116013032980784837094738356175991364395455204835324455810814055944764109234129010492269581408600009386595427991513236458464354768157315483091898970879300954540175247825718514107084608264564889098214264863604883438961600216645976532706988513244819161793096143681897379315082134265617697635800727770233591268184387676917842275673893483582432877323662
e : 65537
n : 23952937352643527451379227516428377705004894508566304313177880191662177061878993798938496818120987817049538365206671401938265663712351239785237507341311858383628932183083145614696585411921662992078376103990806989257289472590902167457302888198293135333083734504191910953238278860923153746261500759411620299864395158783509535039259714359526738924736952759753503357614939203434092075676169179112452620687731670534906069845965633455748606649062394293289967059348143206600765820021392608270528856238306849191113241355842396325210132358046616312901337987464473799040762271876389031455051640937681745409057246190498795697239
##### PRODUCE THE FOLLOWING ####
plaintext
```

We know: `d=e^-1 mod toitent(n)`

We need to find `q`. Then calculate `toitent(n)` to find `d`. 

We know `m = ciphertext^d mod n` so we have `pow(ciphertext,d,n)`


```
IS THIS POSSIBLE and FEASIBLE? (Y/N):y
#### TIME TO SHOW ME WHAT YOU GOT! ###
plaintext: 14311663942709674867122208214901970650496788151239520971623411712977119645236321549653782653
Outstanding move!!!


If you convert the last plaintext to a hex number, then ascii, you'll find what you need! ;)
```

Plaintext is in decimal.

### Flag
`picoCTF{wA8_th4t$_ill3aGal..o1c355060}`

### Solver

```python
import binascii
from pwn import *

# Not my initial code, my function wasn't this clean
MMI = lambda A, n,s=1,t=0,N=0: (n < 2 and t%N or MMI(n, A%n, t, s-A//n*t, N or n),-1)[n<1]

r = remote('2019shell1.picoctf.com', 2611)

# Q1
lines = r.recvuntil('IS THIS POSSIBLE and FEASIBLE? (Y/N):')
print lines
q = int([l for l in lines.split('\n') if 'q :' in l][0].split(':')[1].strip(), 10)
p = int([l for l in lines.split('\n') if 'p :' in l][0].split(':')[1].strip(), 10)
r.sendline('Y')
print r.recvuntil('n:')
ans = q * p
print 'Sending: {}'.format(ans)
r.sendline('{}'.format(ans))

# Q2
lines = r.recvuntil('IS THIS POSSIBLE and FEASIBLE? (Y/N):')
print lines
p = int([l for l in lines.split('\n') if 'p :' in l][0].split(':')[1].strip(), 10)
n = int([l for l in lines.split('\n') if 'n :' in l][0].split(':')[1].strip(), 10)
r.sendline('Y')
print r.recvuntil('q:')
ans = n / p
print 'Sending: {}'.format(ans)
r.sendline('{}'.format(ans))

# Q3
lines = r.recvuntil('IS THIS POSSIBLE and FEASIBLE? (Y/N):')
print lines
r.sendline('N')

# Q4
lines = r.recvuntil('IS THIS POSSIBLE and FEASIBLE? (Y/N):')
print lines
q = int([l for l in lines.split('\n') if 'q :' in l][0].split(':')[1].strip(), 10)
p = int([l for l in lines.split('\n') if 'p :' in l][0].split(':')[1].strip(), 10)
r.sendline('Y')
print r.recvuntil('totient(n):')
ans = (q - 1) * (p - 1)
print 'Sending: {}'.format(ans)
r.sendline('{}'.format(ans))

# Q5
lines = r.recvuntil('IS THIS POSSIBLE and FEASIBLE? (Y/N):')
print lines
plain = int([l for l in lines.split('\n') if 'plaintext :' in l][0].split(':')[1].strip(), 10)
e = int([l for l in lines.split('\n') if 'e :' in l][0].split(':')[1].strip(), 10)
n = int([l for l in lines.split('\n') if 'n :' in l][0].split(':')[1].strip(), 10)
r.sendline('Y')
print r.recvuntil('ciphertext:')
ans = pow(plain, e, n)
print 'Sending: {}'.format(ans)
r.sendline('{}'.format(ans))

# Q6
lines = r.recvuntil('IS THIS POSSIBLE and FEASIBLE? (Y/N):')
print lines
r.sendline('N')

# Q7
lines = r.recvuntil('IS THIS POSSIBLE and FEASIBLE? (Y/N):')
print lines
q = int([l for l in lines.split('\n') if 'q :' in l][0].split(':')[1].strip(), 10)
p = int([l for l in lines.split('\n') if 'p :' in l][0].split(':')[1].strip(), 10)
e = int([l for l in lines.split('\n') if 'e :' in l][0].split(':')[1].strip(), 10)
r.sendline('Y')
print r.recvuntil('d:')
ans = MMI(e, (q - 1) * (p - 1))
print 'Sending: {}'.format(ans)
r.sendline('{}'.format(ans))

# Q8
lines = r.recvuntil('IS THIS POSSIBLE and FEASIBLE? (Y/N):')
print lines
p = int([l for l in lines.split('\n') if 'p :' in l][0].split(':')[1].strip(), 10)
cipher = int([l for l in lines.split('\n') if 'ciphertext :' in l][0].split(':')[1].strip(), 10)
e = int([l for l in lines.split('\n') if 'e :' in l][0].split(':')[1].strip(), 10)
n = int([l for l in lines.split('\n') if 'n :' in l][0].split(':')[1].strip(), 10)
r.sendline('Y')
print r.recvuntil('plaintext:')
q = n / p
d = MMI(e, (q - 1) * (p - 1))
ans = pow(cipher, d, n)
print 'Sending: {}'.format(ans)
r.sendline('{}'.format(ans))
lines = r.recvall()
print lines
print 'In hex: {}'.format(hex(ans))
print binascii.unhexlify(hex(ans)[2:])
```