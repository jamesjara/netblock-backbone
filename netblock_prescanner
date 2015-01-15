<?php
/* nirsoft managment

//Webservice
scanNetBlock(...);
updateNetBlock(..);
addIpStatus(...);
*/
error_reporting(0);ini_set('display_errors', 0);

class nirsoftBackbone extends SQLite3{ 
    function __construct() {
        $this->open('nirsoft');
    }
}

$db = new nirsoftBackbone();

function execute($data){
    //sleep(2000);
    exec(sprintf('ping -c 1 %s 2>&1', $data), $output, $return_var);
    $stdout = implode($output," ");
    if(strpos($stdout,'unknown') !== false){
        return $stdout;
    }
    return true;
}

function updateNetBlock($country,$start_ip, $end_ip){
    global $db;
    $db->query(sprintf('UPDATE netblocks set status = 1 where country = "%s" and start_ip = "%s" and end_ip  = "%s"', $country, $start_ip, $end_ip));
}

function addIpStatus($address, $type ,$elapse_time , $now, $status=0){
    global $db;
    $db->query(sprintf('insert into ipsLogs values("%s","%s","%s","%s","%s");', $address, $type ,$elapse_time , $now, $status));
}

function elapse($performance_start){
    return microtime(true)-$performance_start;
}

/**
Returns unscanned netblock by country.
**/
function scanNetBlock($country, $mask=null){
    global $db;
    $debug = 'limit 1'; //To test
    $queqe = array();
    $netblocks = array();
    $isWild = ($mask!=null) ? true : false;
    $msg_success = ($isWild) ? 'DONE-RANGE: %s in %s' : 'DONE-SINGLE: %s in %s';
    $msg_error = ($isWild) ? 'ERROR-RANGE: %s in %s' : 'ERROR-SINGLE: %s in %s';
    $results = $db->query(sprintf('SELECT * FROM netblocks where country = "%s" and (status <> 1 or status is null) '.$debug, $country));
    if($isWild){
        while ($row = $results->fetchArray()) {
           $queqe[] = sprintf('%s/%s', $row["start_ip"], $mask);
           $netblocks[] = array($row["start_ip"], $row["end_ip"]);
        }
    } else {
        while ($row = $results->fetchArray()) {
           for ($ip = ip2long($row["start_ip"]); $ip<=ip2long($row["end_ip"]); $ip++){
               $queqe[] = sprintf('%s', long2ip($ip));
           }
           $netblocks[] = array($row["start_ip"], $row["end_ip"]);
        }
    }
    foreach($queqe as $address){
        $performance_start = microtime(true);
        $exec_result = execute($address);
        $elapse_time = elapse($performance_start);
        $status = 0;
        if($exec_result===True){
            echo sprintf($msg_success ,$address, $elapse_time)."\xA"; //cant use printf cause jumpline
        } else {
            $status = 1;
            echo sprintf($msg_error, $address, $elapse_time)."\xA";
            //TODO create errors table
        }
        addIpStatus($address, (($isWild) ? 'wild' : 'common' ) ,$elapse_time , date('Y-m-d H:i:s'), $status);
    }
    foreach($netblocks as $row){
        updateNetBlock($country, $row[0], $row[1]);
    }
}

//CLI-MODE
if (($argc > 1)==false) {
    echo "\xA";
    echo "//////////////////////////////////////////////////////"."\xA";
    echo "//                netblock pre scanner              //"."\xA";
    echo "// Usage: php netblock_prescanner <country>         //"."\xA";
    echo "// Usage: php netblock_prescanner <country> <mask>  //"."\xA";
    echo "//////////////////////////////////////////////////////"."\xA";
    exit;
}

$mask=null;
if($argv[2]) $mask=$argv[2];
scanNetBlock($argv[1], $mask);

