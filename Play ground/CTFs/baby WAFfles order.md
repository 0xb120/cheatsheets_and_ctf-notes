---
Category:
  - Web
Difficulty: Easy
Platform: HackTheBox
Status: 3. Complete
tags:
  - JSON2XML
  - XXE-Injection
---
>[!quote]
> Our WAFfles and ice scream are out of this world, come to our online WAFfles house and check out our super secure ordering system API!****


# Set up

-

# Information Gathering

![Pasted image 20210818201502.png](../../zzz_res/attachments/Pasted_image_20210818201502.png)

```php
<?php
class OrderController
{
    public function order($router)
    {
        $body = file_get_contents('php://input');
        if ($_SERVER['HTTP_CONTENT_TYPE'] === 'application/json')
        {
            $order = json_decode($body);
            if (!$order->food) 
                return json_encode([
                    'status' => 'danger',
                    'message' => 'You need to select a food option first'
                ]);
            return json_encode([
                'status' => 'success',
                'message' => "Your {$order->food} order has been submitted successfully."
            ]);
        }
        else if ($_SERVER['HTTP_CONTENT_TYPE'] === 'application/xml')
        {
            $order = simplexml_load_string($body, 'SimpleXMLElement', LIBXML_NOENT);
            if (!$order->food) return 'You need to select a food option first';
            return "Your {$order->food} order has been submitted successfully.";
        }
        else
        {
            return $router->abort(400);
        }
    }
}
```

![Pasted image 20210818201919.png](../../zzz_res/attachments/Pasted_image_20210818201919.png)

# The Bug

Back-end also accepts XML instead of JSON, so it is possible to inject XXE inside XML and the server will execute it.

From:

```json
{"table_num":"1","food":"WAFfles"}
```

To:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<root>
   <food>WAFfles</food>
   <table_num>1</table_num>
</root>
```

# Exploitation

![Pasted image 20210818202015.png](../../zzz_res/attachments/Pasted_image_20210818202015.png)

![Pasted image 20210818202326.png](../../zzz_res/attachments/Pasted_image_20210818202326.png)

![Pasted image 20210818202442.png](../../zzz_res/attachments/Pasted_image_20210818202442.png)

# Flag

>[!success]
>`HTB{wh0_l3t_th3_XX3_0ut??w00f..w00f..w00f..WAFfles!}`

