# Отправка электронной почты с использованием SMTP-сервера Gmail из скрипта PHP

If we have a website then we obviously need to send emails to users. That email should go through your contact us page, through your newsletter, on user registration etc.

PHP provides a `mail()` function which used to send an email. But there are limitations while using `mail()` method. You can’t send email from a local development server. Another drawback is, there is a high possibility of your email ended up into a Spam folder.

To get out of these problems, we need to use SMTP server to send emails.

In this article, we study how to use [PHPMailer](https://github.com/PHPMailer/PHPMailer) and Gmail SMTP server for sending an emails.

### Installation

We first need to install PHPMailer library in our project. Recommended way to install library is through [Composer](https://getcomposer.org/) .

Open the command prompt in your project root directory and run the below command.

composer require phpmailer/phpmailer

As we are using Gmail SMTP, we need to change some settings on our Google account. Login to your Google account and click on My Account. Once you are on My Account page then click on Security. Scroll down to the bottom and you will find ‘Less secure app access’ settings. Set it to ON.

 ![less-secure-apps](/images/d89746888da2d9510b64a9f031eaecd5.gif)   ![less-secure-apps](/images/3349fb712a42891ae4c902a9d79fff2f.png)  

Next, we need to write a code using PHPMailer library along with Gmail SMTP server settings.

### PHP Script For Sending Email Using Gmail SMTP Server

Open your PHP file where you need to write a code for emails. For instance, we are assuming you have `sendemail.php` file in the root directory.

 **sendemail.php** 

\<?php
//Import PHPMailer classes into the global namespace
use PHPMailer\\PHPMailer\\PHPMailer;
use PHPMailer\\PHPMailer\\Exception;

require\_once 'vendor/autoload.php';

$mail = new PHPMailer(true);
?>

In the above code, we have included the environment of the PHPMailer library in our file. Next, for sending emails using PHPMailer we need to pass Gmail SMTP server address, SMTP port for Gmail and SMTP authentication(which is nothing but your username and password of a Google account).

$mail->isSMTP();
$mail->Host = 'smtp.googlemail.com';  //gmail SMTP server
$mail->SMTPAuth = true;
$mail->Username = 'GMAIL\_USERNAME';   //username
$mail->Password = 'GMAIL\_PASSWORD';   //password
$mail->SMTPSecure = 'ssl';
$mail->Port = 465;                    //SMTP port

We have setup our Gmail SMTP server settings. Now, we all good to go for sending an email to a user.

$mail->setFrom('FROM\_EMAIL\_ADDRESS', 'FROM\_NAME');
$mail->addAddress('RECEPIENT\_EMAIL\_ADDRESS', 'RECEPIENT\_NAME');

$mail->isHTML(true);

$mail->Subject = 'Email subject';
$mail->Body    = '<b>Email Body</b>';

$mail->send();
echo 'Message has been sent';

### Sending Attachments In An Email

PHPMailer library provides a way for sending single or multiple attachments in an email. We all need to do is pass a directory path of our attachments to the method `addAttachment` .

$mail->addAttachment(\_\_DIR\_\_ . '/attachment1.png');
$mail->addAttachment(\_\_DIR\_\_ . '/attachment2.jpg');

Our final code is as follows.

 **sendemail.php** 

```php
\<?php
use PHPMailer\\PHPMailer\\PHPMailer;
use PHPMailer\\PHPMailer\\Exception;

require\_once "vendor/autoload.php";
require\_once "constants.php";

$mail = new PHPMailer(true);

try {
    $mail->isSMTP();
    $mail->Host = 'smtp.googlemail.com';  //gmail SMTP server
    $mail->SMTPAuth = true;
    $mail->Username = GMAIL\_USERNAME;   //username
    $mail->Password = GMAIL\_PASSWORD;   //password
    $mail->SMTPSecure = 'ssl';
    $mail->Port = 465;                    //smtp port
 
    $mail->setFrom('FROM\_EMAIL\_ADDRESS', 'FROM\_NAME');
    $mail->addAddress('RECEPIENT\_EMAIL\_ADDRESS', 'RECEPIENT\_NAME');

    $mail->addAttachment(\_\_DIR\_\_ . '/attachment1.png');
    $mail->addAttachment(\_\_DIR\_\_ . '/attachment2.png');

    $mail->isHTML(true);
    $mail->Subject = 'Email Subject';
    $mail->Body    = '<b>Email Body</b>';

    $mail->send();
    echo 'Message has been sent';
} catch (Exception $e) {
    echo 'Message could not be sent. Mailer Error: '. $mail->ErrorInfo;
}
?>
```

We hope you understand send email using Gmail SMTP server from a PHP script. You may also like our article [Sending Email Via Gmail SMTP Server In Laravel](https://artisansweb.net/sending-email-via-gmail-smtp-server-laravel) . If you have any questions or suggestions please leave a comment below.

If you liked this article, then please subscribe to our [Youtube Channel](https://www.youtube.com/channel/UCosi8Kv8-EPLt5TBJLlsWJA?sub_confirmation=1) for video tutorials.

**********
[PHP](/tags/PHP.md)
