# The Perl Coders’ Guide to the Red Hat Network Satellite/Spacewalk API

## Intended Audience


The document is intended for system administrators and/or Perl coders who wish to make use of the RedHat RHN Satellite v5.1 Server API.

There may well be better ways to code the examples below and it should be noted that they have not been thoroughly examined for security issues.
# API Introduction



Every Perl script that uses the Frontier::RPC module to access the Satellite API must include some variation on the following code:



    1.	#!/usr/bin/perl
    
    2.	use strict;
    3.	use warnings;
    4.	use Frontier::Client;
    
    5.	my $HOST = ‘satellite.company.com’;
    6.	my $client = new Frontier::Client(url => "http://$HOST/rpc/api");
    7.	my $session = $client->call('auth.login', 'user', 'password');


Although not strictly necessary, lines 2 and 3 will assist you in writing better code and catching otherwise difficult to spot mistakes.

The problem with the above approach is the username and password of a valid (and probably privileged) Satellite user is stored in clear text in each and every one of your scripts.  To avoid this, I create the file /etc/satellite_api.conf (which a filemode of 0600 and owned by root:root) which contains:

satellite.company.com user password

and use a home grown perl module called RHNSession.  RHNSession consists of:



    1.	package RHNSession;
    
    2.	use strict;
    3.	use Exporter;
    4.	use vars qw($VERSION @ISA @EXPORT @EXPORT_OK %EXPORT_TAGS);
    
    5.	$VERSION     = 0.01;
    6.	@ISA         = qw(Exporter);
    7.	@EXPORT      = ();
    8.	@EXPORT_OK   = qw(Session);
    9.	%EXPORT_TAGS = ( DEFAULT => [qw(&Session)] );
    
    10.	sub Session {
    
    11.	open(IN, "/etc/satellite_api.conf") or die "Couldn't open file for reading: $!";
    12.	$_ = <IN>;
    13.	my ($HOST, $USER, $PASS) = (split);
    14.	close(IN);
    
    15.	my $client = new Frontier::Client(url => "http://$HOST/rpc/api");
    16.	my $session = $client->call('auth.login', $USER, $PASS);
    
    17.	return ($client, $session);
    
    18.	}
    
    19.	1;


So your minimal API script is now:



    1.	#!/usr/bin/perl
    
    2.	use strict;
    3.	use warnings;
    4.	use Frontier::Client;
    5.	use RHNSession;
    
    6.	my ($client, $session) = RHNSession::Session;
    7.	$client->call('auth.logout', $session);


This allows you to have your scripts readable by other people (so they can see what’s going on) but keep your username/password secret.
# API Insights

## Troubleshooting


Like Satellite itself, the API is not trivial and can take some time to get your head around.  The best three pieces of advice I can give is always do:




    2.	use strict;
    3.	use warnings;


And 



    4.	use Data::Dumper;


And 



    man Frontier::Client



It’s very difficult to understand why your code isn’t working as expected if:
* Perl isn’t warning you about common mistakes (undeclared variables for example can be cased by typing mistakes);
* You can’t visualise the data you are working with; and
* You don’t know how the Perl module you are using is designed to work.
## Passing Arrays

When you view the API help pages, you will often see descriptions similar to:




    Method: addChildChannels
    Description:
    Add child channels to an activation key. 
    Parameters:
     	string sessionKey 
     	string key 
     	array 
     	string childChannelLabel 
    Returns: int - 1 on success
On first thought, it’s quite reasonable to do thing following:



    1.	#!/usr/bin/perl
    
    2.	use strict;
    3.	use warnings;
    4.	use Frontier::Client;
    5.	use RHNSession;
    
    6.	my ($client, $session) = RHNSession::Session;
    
    7.	my $key = “web_servers”;
    
    8.	my @channels = (“label1” ,”label2”);
    
    9.	$client->call(‘activationkey.addChildChannels’, $session, $key, @channels);


The above code will fail with an unhelpful error message similar to



    Fault returned from XML RPC Server, fault code -1: redstone.xmlrpc.XmlRpcFault: Could not find method addChildChannels in class class com.redhat.rhn.frontend.xmlrpc.activationkey.ActivationKeyHandler


The problem is on line 9.  @channels gets expanded out so the entire line becomes:



    9.	$client->call(‘activationkey.addChildChannels’, $session, $key, 'label1', 'label2');


This is obviously wrong.  Instead of passing an array (or list) you need to pass a reference to the array.  So the correct solution is:



    9.	$client->call(‘activationkey.addChildChannels’, $session, $key, \@channels);
## Passing Structures

In other cases you will see similar descriptions to




    Method: createOrUpdatePath
    Description:
    Create a new file or directory with the given path, or update an existing path on a server. 
    Parameters:
     	string sessionKey 
     	int serverId 
     	boolean isDir - True if the path is a directory, False if it is a file. 
     	string path 
     	struct (path info) 
     	string 'contents' - Contents of the text file. (ignored for directories) 
     	string 'owner' - Owner of the file/directory. 
     	string 'group' - Group name of the file/directory. 
     	string 'permissions' - Octal file/directory permissions (eg: 644) 
     	string "macro-start-delimiter" - Config file macro start delimiter. Use null or empty string to accept the default. (ignored if working with a directory) 
     	string "macro-end-delimiter" - Config file macro end delimiter. Use null or empty string to accept the default. (ignored if working with a directory) 
     	int commitToLocal - 1 if to commit the file to the server's local channel, 0 to commit to sandbox. 
    Returns: struct (config revision) 
     	string "path" 
     	string "type" - "file" or "directory". 
     	string "contents" 
     	string "revision" 
     	string "created" 
     	string "modified" 
     	string "owner" 
     	string "group" 
     	string "permissions" 
     	string "channel" - Channel name. 
     	boolean "binary" - Present for files only. 
     	string "md5" - Present for files only. 
     	string "macro-start-delimiter" - Present for files only. 
     	string "macro-end-delimiter" - Present for files only.
 
The thing to notice here is struct (path info).  The easiest way to implement the API call in Perl is to use a reference to a hash (also called an associative array).



    1.	for my $filename (@ssh_files) {
    2.	    chomp $filename;
    3.	    my $filetype = 0; # Assume the file is not a directory
    4.	    my %fileinfo;
    5.	    my $uid = (stat($filename))[4];
    6.	    my $gid = (stat($filename))[5];
    7.	    $fileinfo{'contents'}    = do { local( *ARGV, $/ );  @ARGV= $filename; <> };
    8.	    $fileinfo{'owner'}       = $client->string(getpwuid($uid));
    9.	    $fileinfo{'group'}       = $client->string(getgrgid($gid));
    10.	    $fileinfo{'permissions'} = $client->string(sprintf "%04o", (stat($filename))[2] & 07777);
    11.	    if (-T $filename or -z $filename) {
    12.	        $client->call('configchannel.createOrUpdatePath', $session, $host, $filename, $client->string($filetype), \%fileinfo);
    13.	    } else {
    14.	        print "Cowardly *not* uploading binary file $filename, upload manually\n";
    15.	    }
    16.	}


The important stuff happens on line 12.  Notice how the hash fileinfo is turned into a hash reference with the backslash.
Another interesting point is the test on line 11.  Currently only text files are supported with this API call.  If you want to upload a binary file, either use the web interface or /usr/bin/rhncfg-manager.
## Receiving Arrays of Structures



Consider the following code.



    1.	#!/usr/bin/perl
    2.	
    3.	use strict;
    4.	use warnings;
    5.	
    6.	use Frontier::Client;
    7.	use Data::Dumper;
    8.	
    9.	use RHNSession;
    10.	
    11.	use constant MAX_TRIES => 10;
    12.	
    13.	my ($client, $session) = RHNSession::Session;
    14.	
    15.	my $systems  = $client->call('system.list_user_systems', $session);
    16.	
    17.	print Dumper $systems;


Line 15 makes the API call and stores the reference to an array of hashes in the scalar variable $systems.  A condensed output of line 17 is below.



    $VAR1 = [
              {
                'last_checkin' => bless( do{\(my $o = '20080509T07:09:02')}, 'Frontier::RPC2::DateTime::ISO8601' ),
                'name' => 'node000',
                'id' => '1000010020'
              },
              {
                'last_checkin' => bless( do{\(my $o = '20080509T06:52:58')}, 'Frontier::RPC2::DateTime::ISO8601' ),
                'name' => 'node001',
                'id' => '1000010021'
              },
    -=snip=-
              {
                'last_checkin' => bless( do{\(my $o = '20080509T07:14:19')}, 'Frontier::RPC2::DateTime::ISO8601' ),
                'name' => 'node099',
                'id' => '1000010184'
              },
              {
                'last_checkin' => bless( do{\(my $o = '20080509T07:00:23')}, 'Frontier::RPC2::DateTime::ISO8601' ),
                'name' => 'node100',
                'id' => '1000010185'
              }
            ];


To examine each entry in the array reference individually, try the following code:



    1.	#!/usr/bin/perl
    2.	
    3.	use strict;
    4.	use warnings;
    5.	
    6.	use Frontier::Client;
    7.	use Data::Dumper;
    8.	
    9.	use RHNSession;
    10.	
    11.	use constant MAX_TRIES => 10;
    12.	
    13.	my ($client, $session) = RHNSession::Session;
    14.	
    15.	my $systems  = $client->call('system.list_user_systems', $session);
    16.	
    17.	for my $sys (@$systems) {
    18.	    print “System ID “.$sys->{‘id’}.” is called “.$sys->{‘name’}.”’\n”;
    19.	}

Which produces output similar to the below.



    System ID 1000010020 is called node000
    System ID 1000010021 is called node001
    -=snip=-
    System ID 1000010184 is called node099
    System ID 1000010185 is called node100

On an aside, did you notice the ISO8601 encoded date in the last_checkin field?  To turn that into something useful, try this code snippet:



    17.	for my $sys (@$systems) {
    18.	    my $date_time = $sys->{'last_checkin'};
    19.	    $date_time = sprintf("%d-%02d-%02d%s%s", unpack('A4A2A2AA8', $date_time->value()));
    20.	    print "System ID ".$sys->{'id'}." is called ".$sys->{'name'}." and last checked in at ".$date_time."\n";
    21.	
    22.	}


Which produces output similar to the below.



    System ID 1000010020 is called node000 and last checked in at 2008-05-09T08:09:01
    System ID 1000010021 is called node001 and last checked in at 2008-05-09T07:52:58
    -=snip=-
    System ID 1000010184 is called node099 last checked in at 2008-05-09T07:50:12
    System ID 1000010185 is called node100 and last checked in at 2008-05-09T08:00:23


Conversely, some API calls require you to provide a date in ISO8601 format.  Try this code to make this happen:



    11.	my $date_time = $client->date_time(strftime("%Y%m%dT%H:%M:%S", localtime(time()+60)));
    12.	$client->call('system.scheduleScriptRun', $session, $sys->{'id'}, "root", "root", TTL, $script, $date_time);
## Sources

RedHat Satellite mailing list (https://www.redhat.com/mailman/listinfo/rhn-satellite-users)


Satellite 108 site (https://rhn-satellite.108.redhat.com/wiki/index.php/Main_Page)

Frontier::RPC site (http://search.cpan.org/~kmacleod/Frontier-RPC-0.07b4/lib/Frontier/Client.pm)

Documentation on the Satellite Server: (https://satellite.example.com/rhn/apidoc/index.jsp)
