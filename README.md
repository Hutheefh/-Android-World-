<?php
/*
 *  Self-hosted sitemaps relay script
 *  (c) 2015-2020 PRO Sitemaps,  https://pro-sitemaps.com
 *
 */

 /*
  * Configuration
  *
  * site_id - your website account ID in PRO Sitemaps service
  * site_url - your website URL
  * sitemap_self_url - the link to sitemap on your domain
  * sitemap_remote_url - the link to sitemap on PRO Sitemaps domain (hostedsitemaps.com)
  *
  */

 $script_config = array(
 	'ps_api' => '20190604',
    'site_id' => '4397819',
    'site_url' => 'https://androidworld80.blogspot.com/',
    'api_key' => 'ps_DJMuMSaG.wxkgE7GXjfgokDB4xPp6gVJeeTDz0Mi0fI0unnwzuQHervbJ',
    'sitemap_self_url' => 'https://androidworld80.blogspot.com/pro-sitemaps-4397819.php',
    'sitemap_remote_url' => 'https://pro-sitemaps.com/api/'
 );

 $sitemap_filename = isset($_GET['sn']) ? preg_replace('#[^a-z\d\.\_\-]#i', '', $_GET['sn']) : 'sitemap.xml';



/*
 * Retrieve remote sitemap
 *
 * Check if cURL library is installed, prepare API request and get response
 *
 */
    if(!function_exists('curl_init')){
        echo 'Error: cURL library not enabled';
        exit;
    }
    $ch = curl_init();
    curl_setopt($ch, CURLOPT_URL, $script_config['sitemap_remote_url']);

    $headers = array(
        'User-Agent: Mozilla/5.0 (compatible; PRO Sitemaps Self-hosted sitemaps script; pro-sitemaps.com) Gecko XML-Sitemaps/1.0',
    );
    $fields = array(
        'method' => 'download_sitemap',
        'api_ver' => $script_config['ps_api'],
        'api_key' => $script_config['api_key'],
        'site_id' => $script_config['site_id'],
        'sitemap_self_url' => $script_config['sitemap_self_url'],
        'sitemap_id' => $sitemap_filename,

        'remote_ip' => $_SERVER['REMOTE_ADDR'],
        'remote_user_agent' => $_SERVER['HTTP_USER_AGENT'],
    );
    curl_setopt($ch, CURLOPT_POST, true);
    curl_setopt($ch, CURLOPT_POSTFIELDS, $fields);

    curl_setopt($ch, CURLOPT_HTTPHEADER, $headers);
    curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);
    curl_setopt($ch, CURLOPT_SSL_VERIFYPEER, false);

    $result = curl_exec($ch);
    $info = curl_getinfo($ch);
    curl_close($ch);

/*
 * Parse API response and send content to client
 *
 */
 	header('PRO-Sitemaps-API-v: '.$script_config['ps_api']);

    if(($info['http_code'] != '200') || strstr($info['content_type'], 'json') ){
        $status_message = '503 Service Temporarily Unavailable';
        header('HTTP/1.1 '.$status_message);
        header('Status: '.$status_message);
        header('Retry-After: 300');
        echo $status_message;
    }else {
        header('Content-Type: ' . $info['content_type']);
        header('Content-Length: ' . strlen($result));
        echo $result;
    }
    exit;
