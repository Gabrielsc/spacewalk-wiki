To understand what the tagging codes mean and how to use them, follow the below link:

[Tagging queries](PostgresTechnicalApproach)




*SAMPLE 1:*



    <!-- PGPORT_2:AS Keyword-->
    
    <!-- PGPORT_3:DECODE()  -->
    
    SELECT  CM.recid recid,
    
            CM.method_name method_name,
    
            DECODE( MT.method_type_name,
    
              'Email', CM.email_address,
    
              'Pager', CM.pager_email) method_target
    
      FROM  rhn_contact_methods CM, rhn_method_types MT
    
     WHERE  CM.method_type_id = MT.recid
    
       AND  CM.contact_id = :user_id
    
    ORDER BY CM.method_name
    
      </query>
    
    </mode>



*SAMPLE 2:*



    <!-- PGPORT_2:AS Keyword -->
    
    <!-- PGPORT_3:DECODE() -->
    
    <!-- PGPORT_2:Rewrite Join -->
    
    SELECT WC.id contact_id,
    
           WC.login contact_login,
    
           CM.recid method_id,
    
           CM.method_name method_name,
    
           DECODE( MT.method_type_name, 
    
             'Email', CM.email_address, 
    
             'Pager', CM.pager_email) method_target
    
      FROM rhn_contact_methods CM, 
    
           web_contact WC,
    
           rhn_method_types MT
    
     WHERE WC.org_id = :org_id
    
       AND CM.contact_id (+) = WC.id
    
       AND CM.method_type_id = MT.recid (+)
    
    ORDER BY WC.login, CM.method_name



*SAMPLE 3:*



    <!-- PGPORT_1:NO Change -->
    
    SELECT CG.recid, 
    
           CG.contact_group_name,
    
           CG.customer_id,
    
           CG.strategy_id,
    
           CG.ack_wait,
    
           CG.rotate_first,
    
           CG.last_update_user,
    
           CG.last_update_date,
    
           CG.notification_format_id
    
      FROM contact_methods CM,
    
           contact_group_members CGM,
    
           contact_groups CG
    
     WHERE CM.recid = CGM.member_contact_method_id
    
       AND CGM.contact_group_id = CG.recid
    
       AND CM.recid = :method_id
    

*SAMPLE 4:*

    <!-- PGPORT_3:DECODE(),AS Keyword,Rewrite Join -->
    SELECT WC.id contact_id,
           WC.login contact_login,
           CM.recid method_id,
           CM.method_name method_name,
           DECODE( MT.method_type_name, 
             'Email', CM.email_address, 
             'Pager', CM.pager_email) method_target
      FROM rhn_contact_methods CM, 
           web_contact WC,
           rhn_method_types MT
     WHERE WC.org_id = :org_id
       AND CM.contact_id (+) = WC.id
       AND CM.method_type_id = MT.recid (+)
    ORDER BY WC.login, CM.method_name
