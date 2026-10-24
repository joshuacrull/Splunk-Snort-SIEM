# Splunk-Snort-SIEM


<h1>Splunk SIEM with Snort IDS</h1>

<h2>Lab Description:</h2>
<p>In this Repository you will find instructions on how to deploy a Splunk SIEM with logs forwarded from a Snort IDS. In walk through lab you will find each step to configure the enviroment.</p>><br />
    ADD COMPLETION PHOTOS

<h2>Installation and configuration of Snort</h2>
<p>In this section we will deploy a linux machine that properly runs Snort</p><br />
<p>Note prerequisites: virtual box installed and a copy of Ubuntu 22.0.04 ISO</p>
<p>Iso can be downloaded here: https://ubuntu.com/download/desktop</p>

1. Create virtual machine with at least 2 CPU and 4 GB RAM <br />
   ![Image Alt Text](images/2023-10-09_17-32.png)
2. Update and Upgrade machine:
   - sudo apt update && upgrade -y
3. Before the next step know your interface name and your IP address range. You will be required to enter them following the next step.
4. Install snort with the following command:
   - sudo apt-get install snort -y<br />
   When promted enter you address range and interface name
5. Now we need to edit the snort configuration file to add the ip range.
   - sudo nano /etc/snort/snort.conf<br />
    ![Image Alt Text](images/snort-conf.png)
6. Note in the snort configuration file view the location for local rules that we will create in future steps:
   - the location of the local rules is: /etc/snort/rules/local.rules
    ![Image Alt Text](images/snort-localrules.png)
8. Test confirguation succesfullness with the following command:
    - sudo snort -T -i enp3s0 -c /etc/snort/snort.conf <br />
    - This command will test your your configuration. If it is correct it will show succesfull confirguation<br />

<section id="snort-rules">
  <h2>Snort Rules</h2>

  <p>
    In this section, we will create custom local rules to help Snort IDS
    identify specific network activity. These rules demonstrate how
    signature-based detection can be used to monitor HTTP requests,
    ICMP traffic, TCP connection attempts, and other network behaviors.
  </p>

  <p>
    A useful tool for learning and building Snort rules is
    <a href="https://snorpy.com/" target="_blank" rel="noopener noreferrer">Snorpy</a>.
    It provides a graphical interface for constructing rule syntax.
    Always validate generated rules against your installed Snort version
    before using them.
  </p>

  <h3>1. Edit the Local Rules File</h3>

  <p>Open the local rules file on the Ubuntu Snort sensor:</p>

  <pre><code class="language-bash">sudo nano /etc/snort/rules/local.rules</code></pre>

  <p>
    Add the rules below to the file. Each rule has a unique SID
    (Signature ID) so it can be identified in Snort alerts.
  </p>

  <h3>2. Configure the Custom Rules</h3>

  <pre><code class="language-text"># Rule 1: Project Nightfall HTTP demo file request
alert tcp 192.168.1.5 any -&gt; 192.168.1.10 80 (msg:"NIGHTFALL HTTP Demo File Request"; flow:to_server,established; http_uri; content:"/downloads/security_patch.exe"; nocase; sid:1000001; rev:1;)

# Rule 2: ICMP ping request to Ubuntu
alert icmp any any -&gt; 192.168.1.5 any (msg:"LOCAL ICMP Echo Request To Ubuntu"; itype:8; sid:1000002; rev:1;)

# Rule 3: TCP SYN packets to Ubuntu
alert tcp any any -&gt; 192.168.1.5 any (msg:"LOCAL TCP SYN To Ubuntu"; flags:S; sid:1000003; rev:1;)

# Rule 4: HTTP request to admin path on Kali
alert tcp any any -&gt; 192.168.1.10 80 (msg:"LOCAL HTTP Admin Path Request"; flow:to_server,established; http_uri; content:"/admin"; nocase; sid:1000004; rev:1;)

# Rule 5: HTTP request for an EXE file on Kali
alert tcp any any -&gt; 192.168.1.10 80 (msg:"LOCAL HTTP EXE File Request"; flow:to_server,established; http_uri; content:".exe"; nocase; sid:1000005; rev:1;)

# Rule 6: HTTP request containing wp-admin
alert tcp any any -&gt; 192.168.1.10 80 (msg:"LOCAL HTTP WordPress Admin Path"; flow:to_server,established; http_uri; content:"wp-admin"; nocase; sid:1000006; rev:1;)

# Rule 7: UDP traffic from Ubuntu to DNS port
alert udp 192.168.1.5 any -&gt; any 53 (msg:"LOCAL Ubuntu DNS Traffic"; sid:1000007; rev:1;)

# Rule 8: SSH connection attempt to Ubuntu
alert tcp any any -&gt; 192.168.1.5 22 (msg:"LOCAL SSH Connection Attempt"; flags:S; sid:1000008; rev:1;)

# Rule 9: HTTP URI containing a path traversal pattern
alert tcp any any -&gt; 192.168.1.10 80 (msg:"LOCAL Possible HTTP Path Traversal"; flow:to_server,established; http_uri; content:"../"; sid:1000009; rev:1;)

# Rule 10: HTTP POST request to Kali
alert tcp any any -&gt; 192.168.1.10 80 (msg:"LOCAL HTTP POST Request"; flow:to_server,established; http_method; content:"POST"; sid:1000010; rev:1;)</code></pre>

  <h3>3. Save the Rules</h3>

  <ol>
    <li>Press <kbd>Ctrl</kbd> + <kbd>O</kbd> to save.</li>
    <li>Press <kbd>Enter</kbd> to confirm the filename.</li>
    <li>Press <kbd>Ctrl</kbd> + <kbd>X</kbd> to exit Nano.</li>
  </ol>

  <h3>4. Validate the Rules</h3>

  <p>
    For Snort 3, validate the rules using your actual configuration path.
    The following example assumes the configuration is located at
    <code>/etc/snort/snort.lua</code>:
  </p>

  <pre><code class="language-bash">sudo snort -T \
  -c /etc/snort/snort.lua \
  -R /etc/snort/rules/local.rules</code></pre>

  <p>
    Review the command output for configuration or rule errors before
    starting Snort. Snort 2 and Snort 3 use different configuration
    conventions, so the command and rule compatibility must be checked
    for the installed version.
  </p>

  <h3>5. Rule Overview</h3>

  <table>
    <thead>
      <tr>
        <th>SID</th>
        <th>Detection</th>
        <th>Purpose</th>
      </tr>
    </thead>
    <tbody>
      <tr>
        <td>1000001</td>
        <td>Nightfall demo file request</td>
        <td>Detects the specified HTTP URI from Ubuntu to Kali.</td>
      </tr>
      <tr>
        <td>1000002</td>
        <td>ICMP echo request</td>
        <td>Monitors ping requests directed at Ubuntu.</td>
      </tr>
      <tr>
        <td>1000003</td>
        <td>TCP SYN</td>
        <td>Observes initial TCP connection attempts to Ubuntu.</td>
      </tr>
      <tr>
        <td>1000004</td>
        <td>Admin path</td>
        <td>Flags HTTP URIs containing <code>/admin</code>.</td>
      </tr>
      <tr>
        <td>1000005</td>
        <td>EXE file request</td>
        <td>Flags

<section id="next-steps">
  <h2>Next Steps: Testing Project Nightfall</h2>

  <p>
    After adding the local rules, the next phase is to validate the Snort
    configuration, generate controlled traffic between the Ubuntu sensor
    and Kali Purple server, and verify that the expected alerts are produced.
    The lab uses a harmless text file for all testing.
  </p>

  <h3>Step 1: Verify the VirtualBox Lab Network</h3>

  <p>
    Start both virtual machines and verify that each system has the
    correct IP address on the internal lab network.
  </p>

  <p><strong>On Ubuntu (Snort sensor):</strong></p>

  <pre><code class="language-bash">ip -br addr
ping -c 4 192.168.1.10</code></pre>

  <p><strong>On Kali Purple (HTTP server):</strong></p>

  <pre><code class="language-bash">ip -br addr
ping -c 4 192.168.1.5</code></pre>

  <p>
    Confirm that Ubuntu uses <code>192.168.1.5/24</code> and Kali uses
    <code>192.168.1.10/24</code> on their lab interfaces. Both adapters
    should be attached to the same VirtualBox Internal Network,
    <code>NightfallNet</code>.
  </p>

  <p>
    <strong>Expected result:</strong> The VMs can communicate over the
    internal network. If the ping test fails, verify the adapter settings,
    IP addresses, and guest firewall rules before proceeding.
  </p>

  <h3>Step 2: Verify the Snort Version and Configuration</h3>

  <p>On Ubuntu, confirm that Snort 3 is installed:</p>

  <pre><code class="language-bash">snort -V</code></pre>

  <p>
    Identify the actual Snort configuration file. Common paths include
    <code>/etc/snort/snort.lua</code> and
    <code>/usr/local/etc/snort/snort.lua</code>.
  </p>

  <pre><code class="language-bash">sudo find /etc /usr/local -name snort.lua 2&gt;/dev/null</code></pre>

  <p>
    Use the configuration path returned by your system in the following
    steps. The example commands below assume
    <code>/etc/snort/snort.lua</code>.
  </p>

  <h3>Step 3: Validate the Local Rules</h3>

  <p>Run a Snort configuration test before monitoring traffic:</p>

  <pre><code class="language-bash">sudo snort -T \
  -c /etc/snort/snort.lua \
  -R /etc/snort/rules/local.rules</code></pre>

  <p>
    <strong>Expected result:</strong> Snort finishes its configuration
    test without fatal errors and accepts the loaded rules. Review the
    output carefully; do not proceed if the rules fail to load.
  </p>

  <p>
    If your rules are stored in another directory, replace the rule path.
    Snort 3 and Snort 2 have different configuration conventions, so
    verify your version before using these commands.
  </p>

  <h3>Step 4: Create the Harmless Demo File on Kali</h3>

  <p>Create the directory that will hold the demonstration file:</p>

  <pre><code class="language-bash">mkdir -p ~/nightfall-web/downloads</code></pre>

  <p>Create a plain-text file with the requested filename:</p>

  <pre><code class="language-bash">printf 'Project Nightfall harmless training artifact.\nThis is plain text, not executable malware.\n' \
  &gt; ~/nightfall-web/downloads/security_patch.exe</code></pre>

  <p>Verify that the file contains plain text:</p>

  <pre><code class="language-bash">file ~/nightfall-web/downloads/security_patch.exe
cat ~/nightfall-web/downloads/security_patch.exe</code></pre>

  <p>
    <strong>Expected result:</strong> The file is identified as text and
    displays the harmless training message. The <code>.exe</code> extension
    is used only to simulate a file-request scenario.
  </p>

  <h3>Step 5: Start the HTTP Server on Kali</h3>

  <p>Run the following command in a Kali terminal:</p>

  <pre><code class="language-bash">sudo python3 -m http.server 80 \
  --bind 192.168.1.10 \
  --directory "$HOME/nightfall-web"</code></pre>

  <p>
    Keep this terminal open. The Python server should listen on TCP port 80
    and display incoming HTTP requests in its terminal output.
  </p>

  <p>
    If the command reports that port 80 is already in use, investigate
    which process owns that port before making changes.
  </p>

  <h3>Step 6: Test HTTP Connectivity from Ubuntu</h3>

  <p>On Ubuntu, request the demo file:</p>

  <pre><code class="language-bash">curl -v \
  http://192.168.1.10/downloads/security_patch.exe \
  -o /tmp/security_patch.exe</code></pre>

  <p>Verify the downloaded file:</p>

  <pre><code class="language-bash">file /tmp/security_patch.exe
cat /tmp/security_patch.exe</code></pre>

  <p>
    <strong>Expected result:</strong> Curl receives an HTTP response and
    saves the harmless text file. The Kali server terminal should show
    a request for
    <code>/downloads/security_patch.exe</code>.
  </p>

  <p>
    If this test fails, troubleshoot connectivity and the HTTP server
    before testing Snort.
  </p>

  <h3>Step 7: Identify the Ubuntu Lab Interface</h3>

  <p>Run the following command on Ubuntu:</p>

  <pre><code class="language-bash">ip -br addr</code></pre>

  <p>
    Identify the interface assigned <code>192.168.1.5</code>. It might
    be named <code>enp0s8</code>, <code>enp0s3</code>, or something else.
    Use the interface name from your actual output.
  </p>

  <h3>Step 8: Start Snort in IDS Alert Mode</h3>

  <p>
    On Ubuntu, start Snort with the custom rules loaded. Replace the
    example interface name if necessary.
  </p>

  <pre><code class="language-bash">sudo snort \
  -c /etc/snort/snort.lua \
  -R /etc/snort/rules/local.rules \
  -i enp0s8 \
  -A alert_fast</code></pre>

  <p>
    Leave Snort running in the foreground. This command configures
    detection and alerting; it does not block traffic.
  </p>

  <h3>Step 9: Trigger the Project Nightfall Detection</h3>

  <p>
    Open a second Ubuntu terminal while Snort is running. Make the target
    HTTP request again:
  </p>

  <pre><code class="language-bash">curl -v \
  http://192.168.1.10/downloads/security_patch.exe \
  -o /tmp/security_patch.exe</code></pre>

  <p>
    Review the Snort terminal for the alert:
    <code>NIGHTFALL HTTP Demo File Request</code>.
  </p>

  <p>
    <strong>Expected result:</strong> If the custom rule is loaded correctly
    and Snort sees and inspects the HTTP request, the matching request
    should produce the custom alert.
  </p>

  <p>
    A successful curl request alone does not prove that Snort detected
    the traffic. Confirm the actual alert before recording the test as
    successful.
  </p>

  <h3>Step 10: Test a Non-Matching Request</h3>

  <p>With Snort still running, request the root page:</p>

  <pre><code class="language-bash">curl -v http://192.168.1.10/ \
  -o /tmp/nightfall-index.html</code></pre>

  <p>
    <strong>Expected result:</strong> The custom Nightfall rule should not
    alert on this request because the URI does not contain
    <code>/downloads/security_patch.exe</code>. Other rules may generate
    alerts for unrelated traffic.
  </p>

  <p>
    Repeat the target file request afterward to confirm that the matching
    request produces the expected result consistently.
  </p>

  <h3>Step 11: Troubleshoot Missing Alerts</h3>

  <p>If Snort does not produce the expected alert:</p>

  <ol>
    <li>Confirm that curl succeeds and the Kali server logs the request.</li>
    <li>Confirm that Snort is listening on the interface assigned <code>192.168.1.5</code>.</li>
    <li>Repeat the Snort configuration test and confirm the rule file loads.</li>
    <li>Check the terminal for syntax errors or missing configuration files.</li>
    <li>Use packet capture to verify that the Ubuntu interface sees the HTTP traffic.</li>
  </ol>

  <p>For packet-level troubleshooting, install tcpdump if needed:</p>

  <pre><code class="language-bash">sudo apt install -y tcpdump</code></pre>

  <p>Then monitor the actual lab interface:</p>

  <pre><code class="language-bash">sudo tcpdump -ni enp0s8 \
  'host 192.168.1.10 and tcp port 80'</code></pre>

  <p>
    Replace <code>enp0s8</code> with your actual interface name. Run the
    curl request in a separate terminal and check whether packets appear.
  </p>

  <h3>Step 12: Capture Genuine Evidence</h3>

  <p>
    Save screenshots of your own system output as you complete the lab.
    Do not create screenshots of expected output and present them as
    verified results.
  </p>

  <ul>
    <li>VirtualBox network settings and both VM IP addresses.</li>
    <li>Snort 3 version output.</li>
    <li>Local rules file.</li>
    <li>Successful Snort configuration test.</li>
    <li>Harmless demo file and its file-type output.</li>
    <li>Curl output and the matching Kali HTTP server log.</li>
    <li>The actual Project Nightfall Snort alert.</li>
    <li>The non-matching URI test.</li>
  </ul>

  <h3>Step 13: Document Results on GitHub</h3>

  <p>
    Update the project's README and test-results document with the
    commands used, observed results, screenshots, and troubleshooting
    notes.
  </p>

  <p>
    Record each test as <strong>Passed</strong>, <strong>Failed</strong>,
    or <strong>Not Yet Tested</strong>. Only mark the detection test as
    passed after reviewing an actual Snort alert that matches the
    target request.
  </p>

  <h3>MITRE ATT&amp;CK Mapping</h3>

  <p>
    The demonstration is related conceptually to
    <a href="https://attack.mitre.org/techniques/T1105/" target="_blank" rel="noopener noreferrer">
      MITRE ATT&amp;CK T1105 — Ingress Tool Transfer
    </a>.
    This is a controlled training scenario. Detecting an HTTP request
    for a filename does not establish that a malicious tool was
    transferred or that the system was compromised.
  </p>

  <h3>Final Lab Status</h3>

  <p>
    Project Nightfall is complete when the VMs communicate correctly,
    Snort accepts the custom rule, the HTTP server delivers the harmless
    file, the matching request generates the expected alert, the
    non-matching request is compared, and the results are documented
    with genuine evidence.
  </p>
</section>






