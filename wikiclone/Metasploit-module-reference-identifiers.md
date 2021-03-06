A reference in a Metasploit module is a source of information related to the module. This can be a link to the vulnerability advisory, a news article, a blog post about a specific technique the module uses, a specific tweet, etc. The more you have the better. However, you should not use this as a form of advertisement.

## List of supported reference identifiers ##

ID  | Source | Code Example
------------- | ------------- | -------------
CVE  | cvedetails.com | [ 'CVE', '2014-9999' ]
OSVDB | osvdb.org | ['OSVDB', '94981']
CWE | cwe.mitre.org | ['CWE', '90']
BID | securityfocus.com | ['BID', '1234']
MSB | technet.microsoft.com | ['MSB', 'MS13-055']
EDB | exploit-db.com | ['EDB', '1337']
US-CERT-VU | kb.cert.org | ['US-CERT-VU', '800113']
ZDI | zerodayinitiative.com | ['ZDI', '10-123']
WPVDB | wpvulndb.com | ['WPVDB', '7615']
URL | anything | ['URL', 'http://example.com/blog.php?id=123']

## Code Example of having references in a module ##

```ruby
require 'msf/core'

class Metasploit3 < Msf::Exploit::Remote
  Rank = NormalRanking

  def initialize(info={})
    super(update_info(info,
      'Name'           => "Code Example",
      'Description'    => %q{
        This is an example of a module using references
      },
      'License'        => MSF_LICENSE,
      'Author'         => [ 'Unknown' ],
      'References'     =>
        [
          [ 'CVE', '2014-9999' ],
          ['OSVDB', '94981'],
          ['BID', '1234'],
          ['URL', 'http://example.com/blog.php?id=123']
        ],
      'Platform'       => 'win',
      'Targets'        =>
        [
          [ 'Example', { 'Ret' => 0x41414141 } ]
        ],
      'Payload'        =>
        {
          'BadChars' => "\x00"
        },
      'Privileged'     => false,
      'DisclosureDate' => "Apr 1 2014",
      'DefaultTarget'  => 0))
  end

  def exploit
    print_debug('Hello, world')
  end

end
```