# Issabel 4, Call Center Module and SipML5 Integration
(Note:$onlyCallback=1; Module working with only callback mode agents)

Thanks:

https://github.com/DoubangoTelecom/sipml5

https://www.issabel.org/

### Firstly you have install Issabel 4 with Asterisk 11-13-16-18 and Call Center module.
#### After, you need to update with this command.
``` 
yum update
```
### We will need a domain name and we should make a secure certificate with letsencrypt.

#### Copy the cert.pem and privkey.pem files created with letsencrypt to the etc/asterisk/keys folder.
```
cp /etc/letsencrypt/live/yourdomain.com/cert.pem /etc/asterisk/keys
cp /etc/letsencrypt/live/yourdomain.com/privkey.pem /etc/asterisk/keys

cd /etc/asterisk/keys

chown asterisk:asterisk cert.pem
chown asterisk:asterisk privkey.pem
```

### Let's set the Asterisk settings.

![image](https://user-images.githubusercontent.com/8502843/206913257-9b15f119-df84-47f4-814b-ae46c90a291e.png)

![image](https://user-images.githubusercontent.com/8502843/206913516-7977fa38-6950-432d-a8c2-3f04a3df38a1.png)

### Download webphone folder and copy /var/www/html/ inside.

#### We need to make some changes in the /var/www/html/index.php file.
Edit index.php and find this on estimated 340th line
```
themeSetup($smarty, $selectedMenu, $pdbACL, $pACL, $idUser);
```
Add the following lines after this line. 
```
if(strpos($_SERVER[REQUEST_URI], 'agent_console') == true || strpos($_SERVER[REQUEST_URI], 'myex_config') == true)
{
  $webPhoneExtension = $pACL->getUserExtension($_SESSION['issabel_user']);
  $dsn1 = generarDSNSistema('asteriskuser', 'asterisk');
  $pdbACL1 = new paloDB($dsn1);
  $webphonePassword = $pdbACL1->fetchTable("SELECT data from sip WHERE id = '$webPhoneExtension' AND keyword = 'secret'; ")[0];
			
  if($webPhoneExtension>0)
  {
    echo '<script type="text/javascript">';
    echo "localStorage.setItem('mhrgl.com.identity.display_name', $webPhoneExtension);";
    echo "localStorage.setItem('mhrgl.com.identity.impi', $webPhoneExtension);";
    echo "localStorage.setItem('mhrgl.com.identity.impu', 'sip:'+ $webPhoneExtension+'@'+ window.location.hostname);";
    echo "localStorage.setItem('mhrgl.com.identity.password', $webphonePassword[0]);";
    echo "localStorage.setItem('mhrgl.com.identity.realm', window.location.hostname);";
    echo "localStorage.setItem('mhrgl.com.expert.disable_video', 'true');";
    echo "localStorage.setItem('mhrgl.com.expert.disable_callbtn_options', 'true');";
    echo "localStorage.setItem('mhrgl.com.expert.websocket_server_url', 'wss://' + window.location.hostname + ':8089/ws');";
    //echo "localStorage.setItem('mhrgl.com.expert.ice_servers', '[]');";
    echo "localStorage.setItem('mhrgl.com.expert.ice_servers', '[{ url: \'stun:stun.a.google.com:19302\'}]');";
    echo "</script>";
    include("webphone/webphone.php");
  }
}
```
After adding it should look something like this.
![image](https://user-images.githubusercontent.com/8502843/211925339-bf8644a7-48a8-408f-9768-967dee6d4e68.png)

### Add Webrtc Extension
![image](https://user-images.githubusercontent.com/8502843/206914544-894482be-bb15-4dfc-a088-0776c2531840.png)

![image](https://user-images.githubusercontent.com/8502843/211926972-a6abe28a-3c06-4317-9271-819666597c18.png)

### Assing extension for user
![image](https://user-images.githubusercontent.com/8502843/206914468-7b1b3d6e-bbca-4ffd-bb19-ecafd7099b60.png)

### Finally edit /var/www/html/modules/agent_console/index.php file. 
Find  $onlyCallback=0; on line 52 and change like this $onlyCallback=1;
![image](https://user-images.githubusercontent.com/8502843/207910860-5b13c767-1317-431f-944a-f17faf5cfe13.png)


## That's all we're going to do. When we enter agent_console and myex_config pages, the softphone becomes active.
![image](https://user-images.githubusercontent.com/8502843/206915181-1fe237d6-76a3-4413-876a-15accea6e25c.png)

