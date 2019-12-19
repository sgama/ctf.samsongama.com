# PicoCTF19 Waves Lambda

## Challenge

We made alot of substitutions to encrypt this. Can you decrypt it? Connect with `nc 2019shell1.picoctf.com 32282`.

### Hints

Flag is not in the usual flag format

## Solution

```bash
$ nc 2019shell1.picoctf.com 32282
-------------------------------------------------------------------------------
qsfaoimw bpop xw ysco dkia - dopzcpfqy_xw_q_supo_kithei_jmmbmmshcq
-------------------------------------------------------------------------------
hpmlppf cw mbpop liw, iw x biup ikopiey wixe wstplbpop, mbp hsfe sd mbp wpi. hpwxepw bskexfa sco bpiomw msapmbpo mboscab ksfa jpoxsew sd wpjioimxsf, xm bie mbp pddpqm sd tinxfa cw mskpoifm sd piqb smbpo'w yiofwife pupf qsfuxqmxsfw. mbp kilypombp hpwm sd ske dpkkslwbie, hpqicwp sd bxw tify ypiow ife tify uxomcpw, mbp sfky qcwbxsf sf epqn, ife liw kyxfa sf mbp sfky oca. mbp iqqscfmifm bie hoscabm scm ikopiey i hsg sd estxfspw, ife liw msyxfa ioqbxmpqmcoikky lxmb mbp hsfpw. tioksl wim qosww-kpaape oxabm idm, kpifxfa iaixfwm mbp txvvpf-tiwm. bp bie wcfnpf qbppnw, i ypkksl qstjkpgxsf, i wmoixabm hiqn, if iwqpmxq iwjpqm, ife, lxmb bxw iotw eosjjpe, mbp jiktw sd bifew scmlioew, opwpthkpe if xesk. mbp exopqmso, wimxwdxpe mbp ifqbso bie asse bske, tiep bxw liy idm ife wim eslf itsfawm cw. lp pgqbifape i dpl lsoew kivxky. idmpolioew mbpop liw wxkpfqp sf hsioe mbp yiqbm. dso wstp opiwsf so smbpo lp exe fsm hpaxf mbim aitp sd estxfspw. lp dpkm tpexmimxup, ife dxm dso fsmbxfa hcm jkiqxe wmioxfa. mbp eiy liw pfexfa xf i wpopfxmy sd wmxkk ife pgzcxwxmp hoxkkxifqp. mbp limpo wbsfp jiqxdxqikky; mbp wny, lxmbscm i wjpqn, liw i hpfxaf xttpfwxmy sd cfwmixfpe kxabm; mbp upoy txwm sf mbp pwwpg tiowb liw kxnp i aicvy ife oiexifm dihoxq, bcfa dost mbp lssepe oxwpw xfkife, ife eoijxfa mbp ksl wbsopw xf exijbifscw dskew. sfky mbp aksst ms mbp lpwm, hossexfa supo mbp cjjpo opiqbpw, hpqitp tsop wsthop pupoy txfcmp, iw xd ifapope hy mbp ijjosiqb sd mbp wcf.
```

Seems to be some sort of email or letter. Could be any cipher. Let's try our tools:
[https://www.guballa.de/substitution-solver](https://www.guballa.de/substitution-solver)

```
-------------------------------------------------------------------------------
congrats here is your flag - frequency_is_c_over_lambda_ptthttobuc
-------------------------------------------------------------------------------
between us there was, as i have already said somewhere, the bond of the sea. besides holding our hearts together through long periods of separation, it had the effect of making us tolerant of each other's yarnsand even convictions. the lawyerthe best of old fellowshad, because of his many years and many virtues, the only cushion on deck, and was lying on the only rug. the accountant had brought out already a box of dominoes, and was toying architecturally with the bones. marlow sat cross-legged right aft, leaning against the mizzen-mast. he had sunken cheeks, a yellow complexion, a straight back, an ascetic aspect, and, with his arms dropped, the palms of hands outwards, resembled an idol. the director, satisfied the anchor had good hold, made his way aft and sat down amongst us. we exchanged a few words lazily. afterwards there was silence on board the yacht. for some reason or other we did not begin that game of dominoes. we felt meditative, and fit for nothing but placid staring. the day was ending in a serenity of still and exquisite brilliance. the water shone pacifically; the sky, without a speck, was a benign immensity of unstained light; the very mist on the essex marsh was like a gauzy and radiant fabric, hung from the wooded rises inland, and draping the low shores in diaphanous folds. only the gloom to the west, brooding over the upper reaches, became more sombre every minute, as if angered by the approach of the sun.
```

### Flag
`picoCTF{frequency_is_c_over_lambda_ptthttobuc}`
