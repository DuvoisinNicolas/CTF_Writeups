cWe can see multiples files.
In database folder:
- canonical_address.db
```sql
SQLite format 3
/tablecanonical_addressescanonical_addresses
CREATE TABLE canonical_addresses (_id integer PRIMARY KEY, address TEXT NOT NULL)W
ctableandroid_metadataandroid_metadata
CREATE TABLE android_metadata (locale TEXT)
fr_FR
%+33742241337
-Service messages
WAP_PUSH_SI!
```
- jobqueue-SilenceJobs-journal
```sql
SQLite format 3
Gtablequeuequeue
CREATE TABLE queue (_id INTEGER PRIMARY KEY, item TEXT NOT NULL, encrypted INTEGER DEFAULT 0)W
ctableandroid_metadataandroid_metadata
CREATE TABLE android_metadata (locale TEXT)
fr_FR
```

- messages.db
Some tables definitions:
```sql
CREATE TABLE thread (_id INTEGER PRIMARY KEY, date INTEGER DEFAULT 0, message_count INTEGER DEFAULT 0, recipient_ids TEXT, snippet TEXT, snippet_cs INTEG
ER DEFAULT 0, read INTEGER DEFAULT 1, type INTEGER DEFAULT 0, error INTEGER DEFAULT 0, snippet_type INTEGER DEFAULT 0, snippet_uri TEXT DEFAULT NULL, arc
hived INTEGER DEFAULT 0, status INTEGER DEFAULT 0, last_seen INTEGER DEFAULT 0)                                                                          tablepartpart                                                                                                                                            
CREATE TABLE part (_id INTEGER PRIMARY KEY, mid INTEGER, seq INTEGER DEFAULT 0, ct TEXT, name TEXT, chset INTEGER, cd TEXT, fn TEXT, cid TEXT, cl TEXT, c
tt_s INTEGER, ctt_t TEXT, encrypted INTEGER, pending_push INTEGER, _data TEXT, data_size INTEGER, thumbnail TEXT, aspect_ratio REAL, unique_id INTEGER NOT NULL)                                                                                                                                                  
Ktablemmsmms                                                                                                                                             
CREATE TABLE mms (_id INTEGER PRIMARY KEY, thread_id INTEGER, date INTEGER, date_received INTEGER, msg_box INTEGER, read INTEGER DEFAULT 0, m_id TEXT, sub TEXT, sub_cs INTEGER, body TEXT, part_count INTEGER, ct_t TEXT, ct_l TEXT, address TEXT, address_device_id INTEGER, exp INTEGER, m_cls TEXT, m_type INT
EGER, v INTEGER, m_size INTEGER, pri INTEGER, rr INTEGER, rpt_a INTEGER, resp_st INTEGER, st INTEGER, tr_id TEXT, retr_st INTEGER, retr_txt TEXT, retr_tx
t_cs INTEGER, read_status INTEGER, ct_cls INTEGER, resp_txt TEXT, d_tm INTEGER, date_delivery_received INTEGER DEFAULT 0, mismatched_identities TEXT DEFA
ULT NULL, network_failures TEXT DEFAULT NULL,d_rpt INTEGER, subscription_id INTEGER DEFAULT -1, notified INTEGER DEFAULT 0)                              
tablesmssms                                                                                                                                              
CREATE TABLE sms (_id integer PRIMARY KEY, thread_id INTEGER, address TEXT, address_device_id INTEGER DEFAULT 1, person INTEGER, date INTEGER, date_sent 
INTEGER, protocol INTEGER, read INTEGER DEFAULT 0, status INTEGER DEFAULT -1,type INTEGER, reply_path_present INTEGER, date_delivery_received INTEGER DEF
AULT 0,subject TEXT, body TEXT, mismatched_identities TEXT DEFAULT NULL, service_center TEXT, subscription_id INTEGER DEFAULT -1, notified DEFAULT 0)W
ctableandroid_metadataandroid_metadata 
CREATE TABLE android_metadata (locale TEXT)
windexdraft_thread_indexdrafts CREATE INDEX draft_thread_index ON drafts (thread_id)u
indexmms_addresses_mms_id_indexmms_addresses
CREATE INDEX mms_addresses_mms_id_index ON mms_addresses (mms_id)P
mindexarchived_indexthread
CREATE INDEX archived_index ON thread (archived)n
indexthread_recipient_ids_indexthread$
[...]
```

And many base64 values, associated with numbers:
```sql
33742241337                                                                                                                                             
WNQwcTJx0NDhKgbJOre6fn8VCTsLplEwvEAZlYjeaw6aK3JL/zaRetMnuSqJ50cWsa6ZCRv9vNQv2upbW2fXdir/ycKLf8yZZmVYgKo+kv56ddGi5                                        
+33742241337                                                                                                                                             
/BteKBnQJFzjRP7tV+aZqp7Y4+QFzWeQfrCchBL10zX6Sm7I4o0uwdXwxPH3B5ZWMNJ9/zcUR2KD5K+GSigz1rgShInoLitGd1jf6WRhPz5iJVv9MLQqRbR1NX4pGozU+lP7Eg==                 
+33742241337            
xfkj1x6VDR/glsyWwMalYY3rnSVOHxaeu7YbnrcjBuHQJfB7ABxz9Qc/pvtnQanoK0xgr4TQAiZYzcbr2MyZH83TDrM=                                                             
+33742241337                                                                                                                                             
WvgLQOZZuh6ZRKI6Y7zxFMZMSdQyZ2xxE5j+e9EKGz9pVvmrG7ZBmLh10qFbjyOs9TPL6oSPVfIKMVIZpoyKwBToAnRDd5Ryo7Qc/B0FY70gnyN8QD0TKuN+vSszEHmZj8fXO3O2ZAVeDnjtwrmvNm9Zt
O3+2uZkqX7c9Tq9vdUqoY3uR                                                                                                                                 
+3374224                                                                                                                                               
JA/upunPe5P5CPU9994S8eu039OC/32NotNClTtZEdBOoCts+t6MxGcr0tdzEaJyEuax3r13t2FBo6OSH47wwwWujJ4p3iWHdvsNFDCPbqHdGWKknVL5pN10hGw1O4FESRMfTg==                 
+33742241337                                                                                                                                             
WQTkiNewHp6JpzWOxl5HvaDXc+Z29+vVTNz7NPe/AJ+6r5nLecvujdKvWlqpCywEFQeHP6tDqvGk3c0R4wVDhzhqXPK3q2P4kx8Rq22OvZu7Knu9/                                        
+33742241337                                                                                                                                             
z2xbauY32w4png2BgHoSTgXHEFFtET2EiRtiGRphGm6YcEsPaI51KqHPA8ALvlxqbRmUuCI/oFyDYJdm4oKoM53uA+k=                                                             
+3374224133
```

The base64 values seems to be encrypted.

- files/sessions-v2/3:
```bash
EVK/
F>|i
:L(R
&X##
9BDe
        rxB
(#E6
D'c;F
```

- org.smssecure.smssecure_preferences.xml
```xml
<?xml version='1.0' encoding='utf-8' standalone='yes' ?>
<map>                                                                                                                                                    
    <boolean name="pref_always_ask_for_sim_card" value="true" />            
    <int name="last_version_code" value="145" />                                                                                                         
    <boolean name="pref_rating_enabled" value="false" />                    
    <boolean name="pref_show_sent_time" value="true" />                                                                                                  
    <boolean name="pref_screen_security" value="true" />                                                                                                 
    <boolean name="pref_media_download_roaming" value="false" />            
    <boolean name="pref_delivery_report_sms" value="true" />                
    <boolean name="pref_prompted_delivery_reports" value="true" />                                                                                       
    <int name="keyboard_height_portrait" value="685" />                     
    <boolean name="pref_delivery_report_toast_sms" value="true" />          
    <boolean name="pref_incognito_keyboard" value="false" />                                                                                             
    <boolean name="pref_enable_passphrase_temporary" value="true" />                                                                                     
    <boolean name="pref_auto_complete_key_exchange" value="true" />         
    <string name="pref_notification_privacy">all</string>                                                                                                
    <boolean name="pref_trim_threads" value="false" />                      
    <string name="pref_language">zz</string>                                
    <boolean name="pref_timeout_passphrase" value="false" />                                                                                             
    <boolean name="pref_hide_unread_message_divider" value="false" />                                                                                    
    <boolean name="pref_all_sms" value="true" />                           
    <string name="pref_repeat_alerts">0</string>                            
    <boolean name="pref_all_mms" value="true" />                                                                                                         
    <boolean name="pref_disable_emoji_drawer" value="true" />                                                                                            
    <boolean name="pref_wifi_sms" value="false" />                                                                                                       
    <boolean name="pref_system_emoji" value="true" />                
    <string name="pref_key_ringtone">content://settings/system/notification_sound</string>                                                                                                    
    <boolean name="pref_media_download" value="false" />                    
    <boolean name="pref_disable_passphrase" value="false" />                                                                                             
    <string name="pref_trim_length">500</string>                                                                                                         
    <boolean name="is_first_run" value="false" />                           
    <string name="pref_led_color">green</string>                                                                                                         
    <string name="pref_enter_key_type">enter</string>                       
    <string name="pref_theme">light</string>                                
    <boolean name="pref_key_vibrate" value="true" />                        
    <boolean name="permissions_asked" value="true" />         
    <boolean name="pref_key_inthread_notifications" value="true" />                                                                                      
    <string name="pref_led_blink">500,2000</string>                                                                                                      
    <boolean name="pref_key_enable_notifications" value="true" />                              
    <boolean name="pref_prompted_default_sms" value="false" />                                 
</map>
```

- SecureSMS-Preferences
```xml
<?xml version='1.0' encoding='utf-8' standalone='yes' ?>
<map>
    <int name="passphrase_iterations" value="18743" />
    <boolean name="passphrase_initialized" value="true" />
    <string name="master_secret">/HKfpIBcOS94xxk3V+KuwGG8w6JMG67JK30vU9BB+k2LvU7Jk3sGd4j477m6EMi+Xx7LBS7iMUS11xuvx+zDPv8e16k=</string>
    <string name="asymmetric_master_secret_curve25519_private">S50Z7P9sdnhQdr+IVQ9U6ew+iUdTSFx4z+uO+8/F8LB1xytfDqhlz7ISAy3DQUzmVVGOIbI6KzzqTFYm2HtlNBUp3dtE3ujIi2HEQjxZBpwq2a5o</string>
    <string name="encryption_salt">9Be77hAJpDuviHX1s3iL6Q==</string>
    <string name="pref_identity_private_curve25519">I608P8gIzbHkh1A4ulinQhBo0h+mFU8hIjpV5YoRpZWy7IqvdoGlvbwVrUEcGgr0N9lPneHrigLsQv7oE+FQFVCnkv4ij2JBa3WyY1fekpVejN5/</string>
    <string name="mac_salt">QdB3nWZZ1Y7s5JrWq/dzww==</string>
    <string name="asymmetric_master_secret_curve25519_public">BSVqnzyu4pBX7+fWs9oTToNFlMGlZ0e6F7puN7hmjOFG</string>
    <string name="pref_identity_public_curve25519">BUM/3NOyDAI/O1Q7TjHoY2ZrsmPX+vo1DJdxqJnkkhlv</string>
</map>
```
I can see a master secret, and an encryption salt !

- SecureSMS.xml:
```
<?xml version='1.0' encoding='utf-8' standalone='yes' ?>
<map>
    <boolean name="migrated" value="true" />
</map>
```

So, we know tht the app was using the packet "sms-secure", and we got some keys and encrypted sms data.

Also i found this on sms secure github page:
```text
- Private. SMSSecure uses the TextSecure encryption protocol to provide privacy for every message, every time.
```

After some researches, i found this page:
https://github.com/hydrargyrum/breakthesilence

This is a program to decrypt export from Silence app.
I used this to bruteforce the password:
End of runjar:
```
Or download it in current directory, for example with:
        curl -LO https://bouncycastle.org/download/bcprov-jdk15on-165.jar
EOF
        exit 1
fi

if [ $# -ne 1 ]
then
        echo "usage: $0 SILENCE_EXPORT_DIR_PATH" >&2
        exit 64
fi


printf "Password (leave empty if empty): \n"
read passwd

printf "Trying $passwd\n"

echo "$passwd" | java -cp "$BCPROV_JAR:build/breakthesilence.jar" re.indigo.breakthesilence.MasterSecretUtil "$1/shared_prefs/SecureSMS-Preferences.xml"
```

My bash;
```
#!/bin/bash

# Function to run the command and check the return code
run_command() {
    output=$(./run-all.sh ../ silence-backup.json <<< "$1")
    return_code=$?
    return $return_code
}

# Loop through all 5-digit numbers
for ((password=10000; password<=99999; password++)); do
    echo "$password"
    if run_command "$password"; then
        echo "Password found: $password"
        break
    fi
done


```