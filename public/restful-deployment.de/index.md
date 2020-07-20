# RESTful Deployment

Seit Jahren bin ich begeisterter APEX-Fan. Und wie jeder APEX-Fan habe auch ich einen Workspace auf <https://apex.oracle.com.> Hier bietet Oracle die Möglichkeit, APEX kennenzulernen und auch kleine Demo-Projekte zu hosten. Man kann innerhalb seines Workspaces schalten und walten, wie es einem beliebt. Was Oracle hier aber nicht bietet, ist der externe Zugriff auf die Datenbank / das Schema  mit einer IDE oder SQLPlus. Klar, sonst müsste Oracle ja auch den Port des DB-Listeners veröffentlichen.
<!--more-->
Vor kurzem habe ich einen Artikel von Peter Raganitsch gelesen. Link: <http://www.oracle-and-apex.com/streaming-flat-file-data-into-database> Hier beschreibt er, wie er Logfiles per RESTful Service hochlädt und auswertet. Als ich das gelesen habe, kam mir eine Idee. Warum sollte das nicht auch mit meinem Quellcode funktionieren?

Und was soll ich sagen? Es klappt! Mit meinem derzeitigen Lieblingseditor, Sublime Text 3, kann ich nun meinen Quellcode für alle Arten von Stored Procedures (Trigger, Procedures, Functions und Packages) an meinen Workspace, der irgendwo in der Cloud liegt, zum Beispiel auch bei <https://apex.oracle.com> schicken und kompilieren lassen. Aber wie funktioniert das Ganze?

Als erstes erstellen wir einen RESTful-Service. Dazu gehen wir im SQL-Workshop auf RESTful Services und klicken dort auf Create. Nun definieren wir ein RESTful Service Module.

{{< figure src="/images/restful-deployment/2018-02-12-20_34_30-RESTful-Service-Module.png" title="RESTful Service Module" >}}

Dann definieren wir ein Resource Template.

{{< figure src="/images/restful-deployment/2018-02-12-20_34_49-Resource-Template.png" title="Resource Template" >}}

Zu guter Letzt definieren wir einen Resource Handler.

{{< figure src="/images/restful-deployment/2018-02-12-20_35_51-Resource-Handler.png" title="Resource Handler" >}}

Diese Resource Handler wird mit den folgenden Zeilen als Source befüllt.

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

Im Grunde geht es bei diesen paar Zeilen nur darum, das per „Execute Immediate“ der Inhalt der Datei ausgeführt wird. Falls es zu einem Fehler kommt, wird versucht das DB-Objekt zu ermitteln und den entsprechenden Fehler anzuzeigen. Nun müssen wir uns nur noch um den Upload unseres Quellcodes kümmern. Ich habe mich an dieser Stelle für curl entschieden. Ich denke aber auch das es mit wget gehen müsste. Mit dem folgenden Befehl, laden wir den Inhalt der Datei my_stored_procedure.sql in den Workspace my_workspace_name und lassen ihn dort ausführen.

```shell
curl -X POST \
 --header "Content-Type:text/xml;charset=UTF-8" \
 --data-binary @my_stored_procedure.sql \
 https://apex.oracle.com/pls/apex/my_workspace_name/deploy/compile/
```

Um nun mit Sublime Text 3 den Sourcecode direkt hochzuladen und zu kompilieren, legen wir eine Batch-Datei an, die vom Build System von Sublime Text benutzt wird, zum Beispiel als D:\my_rest_deploy.bat.

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

Anschließend legen wir in Sublime Text unter „Tools / Build System / New Build System …“ ein neues Build System mit folgendem Inhalt an.

```json
{
 // Script                       %1          %2           %3=RestURL
 "cmd":["D:/my_rest_deploy.bat","$file_path","$file_name","https://apex.oracle.com/pls/apex/die21/deploy/compile/"],
 "selector": "source.plsql.oracle",
 "shell":"true"
}
```

Und wenn ich nun meinen Code bearbeite und das entsprechende Build System ausgewählt habe, reicht ein einfaches Strg+B, um mein Package oder was auch immer in die Cloud zu bringen. Das Ganze erweist sich als sehr nützlich, wenn man zum Beispiel einen Skill für ALEXA schreiben möchte, aber auf die DB nur per Web Interface zugreifen kann.

{{< figure src="/images/restful-deployment/sublime_build.gif" title="Screenplay" >}}

An dieser Stelle nochmal ein Dankeschön an [Peter](https://twitter.com/PeterRaganitsch) und [KrisRice](https://twitter.com/krisrice), die mich erst auf diese Idee gebracht haben…

