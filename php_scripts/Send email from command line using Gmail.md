
## Send email from command line using Gmail

Using this code you will be able to send email from command line or any shell script. For running this code you will be required a library named "PHPMailer". You can find it here.
[PHPMailer Library](library/PHPMailer.zip)

Following is a syntax of script execution.

```php

php email.php --to=mail@example.com --cc=abc@example.com,xyz@example.com --sub="Mail Subject" --msg="This is a test mail" --attach="/var/www/test.csv"
```

Save the below code in the file named `email.php`

```php

<?php
require_once('lib/PHPMailerAutoload.php');
try {
    $paramArr = getopt(null, ["to:", "cc:", "bcc:", "sub:", "attach:", "msg:"]);
	$cc_from_to = [];
    if (array_key_exists("to", $paramArr) && array_key_exists("msg", $paramArr)) {
        $tmp = explode(',', $paramArr['to']);
		$paramArr['to'] = array_shift($tmp);
		print_r($paramArr['to']);
		if(count($tmp)> 0){
			$cc_from_to = $tmp;
		}		
		if (!isValidEmail($paramArr['to'])) {
            die("Please provide valid recepient email address!");
        }
    } else {
        die('Please provide atleast recepient email and message!');
    }
    $paramArr['cc'] = array_key_exists("cc", $paramArr) ? explode(',', $paramArr['cc']) : array();
    $paramArr['bcc'] = array_key_exists("bcc", $paramArr) ? explode(',', $paramArr['bcc']) : array();
    $paramArr['sub'] = array_key_exists("sub", $paramArr) ? $paramArr['sub'] : '';
    $paramArr['attach'] = array_key_exists("attach", $paramArr) ? explode(',', $paramArr['attach']) : array();
	if(is_array($cc_from_to) && count($cc_from_to)>0){
		$paramArr['cc'] = array_merge($cc_from_to, $paramArr['cc']);
	}
    $data = array(
        'to' => $paramArr['to'],
        'cc' => $paramArr['cc'],
        'bcc' => $paramArr['bcc'],
        'sub' => $paramArr['sub'],
        'attach' => $paramArr['attach'],
        'msg' => $paramArr['msg']
    );
	print_r($data);
    sendMail($data);
} catch (Exception $e) {
    echo 'Exception : ' . $e->getMessage();
}

function isValidEmail($email) {
    return filter_var($email, FILTER_VALIDATE_EMAIL) !== false;
}

function sendMail($data) {
	$email_id = "Gmail_Email_ID";
	$password = "Gmail_Password or Gmail_Application_Password";

    $mail = new PHPMailer;
    $msgHTML = $data['msg'];
    $mail->isSMTP();
    $mail->SMTPDebug = 0;
    $mail->Debugoutput = 'html';
    $mail->Host = 'smtp.gmail.com';
    $mail->Port = 587;
    $mail->SMTPSecure = 'tls';
    $mail->SMTPAuth = true;
    $mail->Username = $email_id;
    $mail->Password = $password ;
    $mail->setFrom($email_id, 'Your Name');
    $mail->addAddress($data['to'], '');
    $mail->addReplyTo($email_id, 'Your Name');
    foreach ($data['cc'] as $email) {
        $mail->addCC($email, ' ');
    }
    foreach ($data['bcc'] as $bcc) {
        $mail->addBCC($bcc, ' ');
    }
    foreach ($data['attach'] as $attach) {
        $mail->addAttachment($attach, substr($attach, strrpos($attach, '/') + 1));
    }
    $mail->Subject = $data['sub'];
    $mail->msgHTML($msgHTML);
    $mail->AltBody = '';
    if (!$mail->send()) {
        echo "Mailer Error: " . $mail->ErrorInfo;
    } else {
        echo 'Success' . '-' . "Message sent! \n\n";
    }
}
```