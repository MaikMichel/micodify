# RESTful Deployment

I have been an enthusiastic APEX fan for years. And like every APEX fan, I also have a workspace on APEX. <https://apex.oracle.com.> Here Oracle offers the possibility to get to know APEX and to host small demo projects. You can switch and manage within your workspace as you like. What Oracle does not offer here, however, is external access to the database / schema with an IDE or SQLPlus. Of course, otherwise Oracle would have to publish the port of the DB listener.
<!--more-->
Recently I read an article by Peter Raganitsch. Link: <http://www.oracle-and-apex.com/streaming-flat-file-data-into-database>    Here he describes how to upload and evaluate log files via RESTful Service. When I read that, I had an idea. Why shouldn’t this also work with my source code?

And what shall I say? It works! With my current favorite editor, Sublime Text 3, I can now send my source code for all kinds of stored procedures (triggers, procedures, functions, and packages) to my workspace, which is somewhere in the cloud, for example, at <https://apex.oracle.com> and have it compiled. But how does it all work?

First we create a RESTful service. In the SQL-Workshop we go to RESTful Services and click on Create. Now we define a RESTful service module.

{{< figure src="/images/restful-deployment/2018-02-12-20_34_30-RESTful-Service-Module.png" title="RESTful Service Module" >}}

Then we define a resource template.

{{< figure src="/images/restful-deployment/2018-02-12-20_34_49-Resource-Template.png" title="Resource Template" >}}

Finally, we define a resource handler.

{{< figure src="/images/restful-deployment/2018-02-12-20_35_51-Resource-Handler.png" title="Resource Handler" >}}

This resource handler is filled with the following lines as source.

```sql
Declare
  v_source_code     clob             := rtrim(wwv_flow_utilities.blob_to_clob(:body),'/');
  v_err_found       boolean          := false;
  v_obj_name        varchar2(1000)   := regexp_substr(lower(dbms_lob.substr(v_source_code, 32000, 1)),
                         'create or replace (function|procedure|package body|package|trigger) ([^ (]*)', 1, 1, null, 2);
Begin
  -- Prepare Header / Output
  owa_util.mime_header('text/plain', true);
  htp.p(''); -- empty

  -- Execute / Compile  Code
  execute immediate v_source_code;
  
  -- Everything is fine, let the user know it
  htp.p('Result: success');
  :status := 200;
Exception
  when others then
    -- Something went wrong
    htp.p('Result: failure');
    htp.p('Object: ' || v_obj_name);
    htp.p('Error:  ' || sqlerrm);

    -- Maybe we get the object out of source
    for cur in (select rownum idx, line, position, text
                  from user_errors
                 where name = upper(v_obj_name)
                 order by sequence)
    loop  
      -- FirstRow, let's print heading
      if cur.idx = 1 then
        htp.p('   LINE | POSITION | TEXT');
        v_err_found := true;
      end if;

      -- Print Message
      htp.p(lpad(cur.line, 7, ' ')||' | '||lpad(cur.position, 8, ' ') ||' | '||cur.text);
    end loop;

    if not v_err_found then
      htp.p(dbms_utility.format_error_backtrace);
      htp.p(v_source_code);
    end if;

    :status := 400;
End;
```

Basically, these few lines are only about the fact that the content of the file is executed by “Execute Immediate”. If an error occurs, the system tries to determine the DB object and displays the corresponding error. Now all we have to do is upload our source code. I chose curl at this point. But I also think that wget should work. With the following command, we load the contents of the file my_stored_procedure. sql into the workspace my_workspace_name and run it there.

```shell
curl -X POST \
 --header "Content-Type:text/xml;charset=UTF-8" \
 --data-binary @my_stored_procedure.sql \
 https://apex.oracle.com/pls/apex/my_workspace_name/deploy/compile/
```

To directly upload and compile the source code with Sublime Text 3, we create a batch file that is used by the build system of Sublime Text, for example as D: \my_rest_deploy. bat.

```batch
@echo off

REM -- For information only
echo File: %2
echo Path: %1
echo Url:  %3
echo

REM -- Change to the directory where the sublime file is
cd %1

REM -- Deploy to https://apex.oracle.com/pls/apex/my_workspace_name/deploy/compile/
curl -X POST --header "Content-Type:text/xml;charset=UTF-8" --data-binary @%2 %3
```

Then we create a new build system with the following content in Sublime Text under “Tools / Build System / New Build System…”.

```json
{
 // Script                       %1          %2           %3=RestURL
 "cmd":["D:/my_rest_deploy.bat","$file_path","$file_name","https://apex.oracle.com/pls/apex/die21/deploy/compile/"],
 "selector": "source.plsql.oracle",
 "shell":"true"
}
```

And if I now edit my code and have chosen the appropriate build system, a simple Ctrl+B is enough to bring my package or whatever into the cloud. This is very useful if you want to write a skill for ALEXA, for example, but can only access the DB via a web interface.

{{< figure src="/images/restful-deployment/sublime_build.gif" title="Screenplay" >}}

The solution is not perfect and you should never do this on a productive environment. Whoever knows this URL is in the worst case master of your database!

At this point I would like to thank [Peter](https://twitter.com/PeterRaganitsch) and [KrisRice](https://twitter.com/krisrice), who brought me to this idea…

