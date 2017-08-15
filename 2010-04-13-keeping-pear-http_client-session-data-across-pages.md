# Keeping PEAR HTTP_Client session data across pages


I just spent three hours banging my head against this problem, so I hope I can
save you the trouble if you're where I was.

I was trying to save an HTTP_Client object in a PHP session so that it would
maintain its session data as it got moved from page to page. The idea was to
log into a website with it and then go to another page where it would be used
by various PHP scripts called by AJAX to perform tasks on the website I'd
logged into. The only problem: I was getting logged out of the 'site every
time I left the script which logged me in. Somehow the HTTP_Client's session
data was being lost simply by my storing it in $_SESSION.

I made sure I was serializing and unserializing:

    $_SESSION['client'] = serialize($client);  
    ...  
    $client = unserialize($_SESSION['client']);  


Still nothing. I scoured the Internet for people who were having similar
problems. The only thing anybody could tell me was "are you remembering to
serialize and unserialize?" I scoured the pitifully scant documentation on
pear.php.net, trying to glean some shred of insight.

Still nothing.

So I decided to brute force the thing and trace the HTTP_Client's code by
hand. After mangling my beautiful PEAR classes with debug code, I noticed line
78 of CookieManager.php:

    function HTTP_Client_CookieManager($serializeSession = false)

"Wait, a minute," I thought, "you've got to be kidding me." I changed that
sucker to:

    function HTTP_Client_CookieManager($serializeSession = true)

And it worked!

Now before you go hacking your PEAR modules, here's the *right* way to do it:

    $manager = new HTTP_Client_CookieManager(true);  
    $client = new HTTP_Client(null, null, $manager);
