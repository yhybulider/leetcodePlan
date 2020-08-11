@ECHO off

IF "%SYSTEM_PWD%"  == "" SET SYSTEM_PWD=MANAGER
IF "%DUMP_FILE%" == "" SET DUMP_FILE=expConverge.dmp
IF "%NET_SERVICE%" == "" SET NET_SERVICE=ATHENADB

IF "%DA_SERVER%" == "" SET DA_SERVER=%COMPUTERNAME%
IF "%DP_SERVER%" == "" SET DP_SERVER=%COMPUTERNAME%

SET TMPFILENAME=impConverge
SET TMPFILE1=%TEMP%\%TMPFILENAME%1.tmp
SET LOGFILE1=%TEMP%\%TMPFILENAME%1.log

ECHO Import Script started... > %LOGFILE1%
ECHO Import Script started...

IF "%1" == "-T" GOTO Tail
IF "%1" == "-t" GOTO Tail
IF NOT "%1" == "" GOTO Usage
GOTO NoTail

:Tail
START "%LOGFILE1%" tail -f %LOGFILE1%

:NoTail

IF "%NET_SERVICE%" == "" SET CONN_STRING=system/%SYSTEM_PWD%
IF NOT "%NET_SERVICE%" == "" SET CONN_STRING=system/%SYSTEM_PWD%@%NET_SERVICE%

ECHO Dropping database users (athena, adas, athena_imp, transfer) ... >> %LOGFILE1%
ECHO Dropping database users (athena, adas, athena_imp, transfer,EAS) ...
ECHO DROP USER athena 		CASCADE;		> %TMPFILE1%
ECHO DROP USER adas 		CASCADE;		>> %TMPFILE1%
ECHO DROP USER athena_imp	CASCADE;		>> %TMPFILE1%
ECHO DROP USER transfer	CASCADE;		>> %TMPFILE1%
ECHO DROP USER REMS_EAS	CASCADE;		>> %TMPFILE1%
ECHO DROP USER EAS	CASCADE;		>> %TMPFILE1%
ECHO EXIT 0; 						>> %TMPFILE1%
SQLPLUS %CONN_STRING% @%TMPFILE1% 			>> %LOGFILE1%
TYPE %LOGFILE1% | %WINDIR%\system32\FIND "ORA-01940" 	> NUL
IF %ERRORLEVEL% == 0 GOTO ErrorDropUsers



ECHO Writing log file to .\impConverge.log >> %LOGFILE1%
ECHO Writing log file to .\impConverge.log
ECHO Importing data for user adas ... %DUMP_FILE% >> %LOGFILE1%
ECHO Importing data for user adas ... %DUMP_FILE%
impdp %CONN_STRING% dumpfile=%DUMP_FILE% directory=expdp_dir logfile=impConverge.log parallel=8

TYPE .\impConverge.log  >> %LOGFILE1%
DEL .\impConverge.log

:SettingComputerNames
ECHO Setting computer for DA master to %DA_SERVER% ... >> %LOGFILE1%
ECHO Setting computer for DA master to %DA_SERVER% ...
IF "%NET_SERVICE%" == "" SET CONN_STRING=adas/adas
IF NOT "%NET_SERVICE%" == "" SET CONN_STRING=adas/adas@%NET_SERVICE%
ECHO WHENEVER SQLERROR EXIT SQL.SQLCODE			> %TMPFILE1%
ECHO DECLARE					 	>> %TMPFILE1%
ECHO   nNr NUMBER;					>> %TMPFILE1%
ECHO   sName VARCHAR(30);				>> %TMPFILE1%
ECHO BEGIN					 	>> %TMPFILE1%
ECHO   nNr := 0;					>> %TMPFILE1%
ECHO   sName := '%DA_SERVER%';				>> %TMPFILE1%
ECHO   BEGIN 						>> %TMPFILE1%
ECHO     SELECT pc_id INTO nNr FROM (			>> %TMPFILE1%
ECHO       SELECT pc_id FROM sc_pc ORDER BY pc_id )	>> %TMPFILE1%
ECHO     WHERE ROWNUM = 1;				>> %TMPFILE1%
ECHO   EXCEPTION					>> %TMPFILE1%
ECHO     WHEN NO_DATA_FOUND THEN NULL;			>> %TMPFILE1%
ECHO   END;					 	>> %TMPFILE1%
ECHO   IF nNr ^> 0 THEN					>> %TMPFILE1%
ECHO     --DBMS_OUTPUT.PUT_LINE ( 'Existing: nNr=' ^|^| TO_CHAR(nNr) );	>> %TMPFILE1%
ECHO     UPDATE sc_pc SET pc_name = sName 		>> %TMPFILE1%
ECHO     WHERE pc_id = nNr;				>> %TMPFILE1%
ECHO     UPDATE sc_instance i				>> %TMPFILE1%
ECHO     SET inst_name = (				>> %TMPFILE1%
ECHO       SELECT c.cl_name ^|^| ' On ' ^|^| p.pc_name	>> %TMPFILE1%
ECHO       FROM sc_class c, sc_pc p			>> %TMPFILE1%
ECHO       WHERE i.cl_id = c.cl_id			>> %TMPFILE1%
ECHO       AND i.pc_id = p.pc_id)			>> %TMPFILE1%
ECHO     WHERE (inst_id) IN (				>> %TMPFILE1%
ECHO       SELECT inst_id FROM sc_instance WHERE pc_id IN (	>> %TMPFILE1%
ECHO         SELECT MIN(pc_id) FROM sc_pc));		>> %TMPFILE1%
ECHO     COMMIT;					>> %TMPFILE1%
ECHO   ELSE					 	>> %TMPFILE1%
ECHO     SELECT adas_sequence.NEXTVAL INTO nNr FROM DUAL;	>> %TMPFILE1%
ECHO     --DBMS_OUTPUT.PUT_LINE ( 'New: nNr=' ^|^| TO_CHAR(nNr) ); 	>> %TMPFILE1%
ECHO     INSERT INTO sc_pc (pc_id, pc_name) 		>> %TMPFILE1%
ECHO     VALUES (nNr,sName);				>> %TMPFILE1%
ECHO     INSERT INTO SC_INSTANCE ( INST_ID, PC_ID, CL_ID, INST_AUTOSTART, INST_RUNNING, INST_CONFTIMEOUT, INST_NAME, INST_PARAM )	>> %TMPFILE1%
ECHO       SELECT adas_sequence.NEXTVAL, nNr, c.cl_id, 0, 0, 6000, c.cl_name ^|^| ' On ' ^|^| p.pc_name, NULL				>> %TMPFILE1%
ECHO       FROM sc_class c, sc_pc p			>> %TMPFILE1%
ECHO       WHERE p.pc_id = nNr AND				>> %TMPFILE1%
ECHO         c.cl_systemstate ^> (				>> %TMPFILE1%
ECHO         SELECT COUNT(inst_id)				>> %TMPFILE1%
ECHO         FROM sc_instance				>> %TMPFILE1%
ECHO         WHERE cl_id = c.cl_id);				>> %TMPFILE1%
ECHO     UPDATE sc_instance				>> %TMPFILE1%
ECHO     SET inst_autostart = 1				>> %TMPFILE1%
ECHO     WHERE pc_id = nNr AND cl_id IN (				>> %TMPFILE1%
ECHO       SELECT cl_id 				>> %TMPFILE1%
ECHO       FROM sc_class				>> %TMPFILE1%
ECHO       WHERE cl_name IN ('PingMan', 'Comms', 'Scheduler', 'SSL'));				>> %TMPFILE1%
ECHO     COMMIT;					>> %TMPFILE1%
ECHO   END IF;					 	>> %TMPFILE1%
ECHO END;						>> %TMPFILE1%
ECHO /							>> %TMPFILE1%
ECHO EXIT 0;    					>> %TMPFILE1%
SQLPLUS %CONN_STRING% @%TMPFILE1% 			>> %LOGFILE1%
IF NOT %ERRORLEVEL% == 0 GOTO ErrorComputerNames

ECHO Setting computer for DP master to %DP_SERVER% ... >> %LOGFILE1%
ECHO Setting computer for DP master to %DP_SERVER% ...
IF "%NET_SERVICE%" == "" SET CONN_STRING=athena/athena
IF NOT "%NET_SERVICE%" == "" SET CONN_STRING=athena/athena@%NET_SERVICE%
ECHO WHENEVER SQLERROR EXIT SQL.SQLCODE			> %TMPFILE1%
ECHO UPDATE ct_available_machine			>> %TMPFILE1%
ECHO SET host_name = '%DP_SERVER%'			>> %TMPFILE1%
ECHO WHERE machineid = (				>> %TMPFILE1%
ECHO   SELECT MIN(machineid)				>> %TMPFILE1%
ECHO   FROM ct_available_machine);			>> %TMPFILE1%
ECHO COMMIT;					 	>> %TMPFILE1%
ECHO EXIT 0; 						>> %TMPFILE1%
SQLPLUS %CONN_STRING% @%TMPFILE1% 			>> %LOGFILE1%
IF NOT %ERRORLEVEL% == 0 GOTO ErrorComputerNames

ECHO Adding user  %USERNAME% for DA GUI if necessary ... >> %LOGFILE1%
ECHO Adding user  %USERNAME% for DA GUI if necessary ...
IF "%NET_SERVICE%" == "" SET CONN_STRING=adas/adas
IF NOT "%NET_SERVICE%" == "" SET CONN_STRING=adas/adas@%NET_SERVICE%
ECHO WHENEVER SQLERROR EXIT SQL.SQLCODE			> %TMPFILE1%
ECHO BEGIN FOR x IN ( SELECT * FROM DUAL WHERE NOT EXISTS ( SELECT 1 FROM adas_users WHERE user_name = '%USERNAME%' ) ) LOOP 	>> %TMPFILE1%
ECHO   INSERT INTO adas_users ( user_name, user_comment ) VALUES ( '%USERNAME%', '' ); 	>> %TMPFILE1%
ECHO END LOOP; END;  	>> %TMPFILE1%
ECHO / 	>> %TMPFILE1%
ECHO BEGIN FOR x IN ( SELECT * FROM DUAL WHERE NOT EXISTS ( SELECT 1 FROM ur_rel_usrgrp WHERE user_name = '%USERNAME%' ) ) LOOP 	>> %TMPFILE1%
ECHO   INSERT INTO ur_rel_usrgrp ( ur_usrgrp_id, user_name, ur_groupname )  	>> %TMPFILE1%
ECHO   SELECT MAX(ur_usrgrp_id)+1, '%USERNAME%', 'AdministratorGroup' 	>> %TMPFILE1%
ECHO   FROM ur_rel_usrgrp; 	>> %TMPFILE1%
ECHO END LOOP; END;  	>> %TMPFILE1%
ECHO / 	>> %TMPFILE1%
ECHO COMMIT;					 	>> %TMPFILE1%
ECHO EXIT 0; 						>> %TMPFILE1%
SQLPLUS %CONN_STRING% @%TMPFILE1% 			>> %LOGFILE1%
IF NOT %ERRORLEVEL% == 0 GOTO ErrorDaUser

ECHO Adding Foreign key for ae alarm data ... >> %LOGFILE1%
ECHO Adding Foreign key for ae alarm data ...
IF "%NET_SERVICE%" == "" SET CONN_STRING=adas/adas
IF NOT "%NET_SERVICE%" == "" SET CONN_STRING=adas/adas@%NET_SERVICE%
ECHO WHENEVER SQLERROR EXIT SQL.SQLCODE			> %TMPFILE1%
ECHO ALTER TABLE ALARM_AEDATA ADD CONSTRAINT "FK_ALARM_AEDATA_AEID" FOREIGN KEY ("AE_ID") REFERENCES ATHENA.CT_ACTIVE_ELEMENT ("ACTIVE_ELEMENTID") ENABLE NOVALIDATE;	>> %TMPFILE1%
ECHO EXIT 0; 						>> %TMPFILE1%
SQLPLUS %CONN_STRING% @%TMPFILE1% 			>> %LOGFILE1%
IF NOT %ERRORLEVEL% == 0 GOTO ErrorAEAlarmData

:Recompile
ECHO Recompiling  ... >> %LOGFILE1%
ECHO Recompiling ...
IF "%NET_SERVICE%" == "" SET CONN_STRING=adas/adas
IF NOT "%NET_SERVICE%" == "" SET CONN_STRING=adas/adas@%NET_SERVICE%
ECHO WHENEVER SQLERROR EXIT SQL.SQLCODE			> %TMPFILE1%
ECHO DECLARE  >> %TMPFILE1%
ECHO    nVerPos NUMBER; >> %TMPFILE1%
ECHO BEGIN >> %TMPFILE1%
ECHO 	-- recompile views >> %TMPFILE1%
ECHO 	DECLARE >> %TMPFILE1%
ECHO 	  sName VARCHAR2(30); >> %TMPFILE1%
ECHO 	  CURSOR csr_objects IS >> %TMPFILE1%
ECHO 	    SELECT object_name FROM user_objects WHERE object_type = 'VIEW' AND status = 'INVALID'; >> %TMPFILE1%
ECHO 	BEGIN >> %TMPFILE1%
ECHO 	  OPEN csr_objects; >> %TMPFILE1%
ECHO 	  LOOP >> %TMPFILE1%
ECHO 	    FETCH csr_objects INTO sName; >> %TMPFILE1%
ECHO 	    EXIT WHEN csr_objects%%NOTFOUND; >> %TMPFILE1%
ECHO 	    BEGIN >> %TMPFILE1%
ECHO 	      EXECUTE IMMEDIATE 'ALTER VIEW ' ^|^| sName ^|^| ' COMPILE'; >> %TMPFILE1%
ECHO 	    EXCEPTION >> %TMPFILE1%
ECHO 	      WHEN OTHERS THEN NULL;>> %TMPFILE1%
ECHO 	    END; >> %TMPFILE1%
ECHO 	  END LOOP; >> %TMPFILE1%
ECHO 	  CLOSE csr_objects; >> %TMPFILE1%
ECHO 	END; >> %TMPFILE1%
ECHO 	-- recompile functions >> %TMPFILE1%
ECHO 	DECLARE >> %TMPFILE1%
ECHO 	  sName VARCHAR2(30); >> %TMPFILE1%
ECHO 	  CURSOR csr_objects IS >> %TMPFILE1%
ECHO 	    SELECT object_name FROM user_objects WHERE object_type = 'FUNCTION' AND status = 'INVALID'; >> %TMPFILE1%
ECHO 	BEGIN >> %TMPFILE1%
ECHO 	  OPEN csr_objects; >> %TMPFILE1%
ECHO 	  LOOP >> %TMPFILE1%
ECHO 	    FETCH csr_objects INTO sName; >> %TMPFILE1%
ECHO 	    EXIT WHEN csr_objects%%NOTFOUND; >> %TMPFILE1%
ECHO 	    BEGIN >> %TMPFILE1%
ECHO 	      EXECUTE IMMEDIATE 'ALTER FUNCTION ' ^|^| sName ^|^| ' COMPILE'; >> %TMPFILE1%
ECHO 	    EXCEPTION >> %TMPFILE1%
ECHO 	      WHEN OTHERS THEN NULL;>> %TMPFILE1%
ECHO 	    END; >> %TMPFILE1%
ECHO 	  END LOOP; >> %TMPFILE1%
ECHO 	  CLOSE csr_objects; >> %TMPFILE1%
ECHO 	END; >> %TMPFILE1%
ECHO 	-- recompile triggers >> %TMPFILE1%
ECHO 	DECLARE >> %TMPFILE1%
ECHO 	  sName VARCHAR2(30); >> %TMPFILE1%
ECHO 	  CURSOR csr_objects IS >> %TMPFILE1%
ECHO 	    SELECT object_name FROM user_objects WHERE object_type = 'TRIGGER' AND status = 'INVALID'; >> %TMPFILE1%
ECHO 	BEGIN >> %TMPFILE1%
ECHO 	  OPEN csr_objects; >> %TMPFILE1%
ECHO 	  LOOP >> %TMPFILE1%
ECHO 	    FETCH csr_objects INTO sName; >> %TMPFILE1%
ECHO 	    EXIT WHEN csr_objects%%NOTFOUND; >> %TMPFILE1%
ECHO 	    BEGIN >> %TMPFILE1%
ECHO 	      EXECUTE IMMEDIATE 'ALTER TRIGGER ' ^|^| sName ^|^| ' COMPILE'; >> %TMPFILE1%
ECHO 	    EXCEPTION >> %TMPFILE1%
ECHO 	      WHEN OTHERS THEN NULL;>> %TMPFILE1%
ECHO 	    END; >> %TMPFILE1%
ECHO 	  END LOOP; >> %TMPFILE1%
ECHO 	  CLOSE csr_objects; >> %TMPFILE1%
ECHO 	END; >> %TMPFILE1%
ECHO 	-- recompile procedures >> %TMPFILE1%
ECHO 	DECLARE >> %TMPFILE1%
ECHO 	  sName VARCHAR2(30); >> %TMPFILE1%
ECHO 	  CURSOR csr_objects IS >> %TMPFILE1%
ECHO 	    SELECT object_name FROM user_objects WHERE object_type = 'PROCEDURE' AND status = 'INVALID'; >> %TMPFILE1%
ECHO 	BEGIN >> %TMPFILE1%
ECHO 	  OPEN csr_objects; >> %TMPFILE1%
ECHO 	  LOOP >> %TMPFILE1%
ECHO 	    FETCH csr_objects INTO sName; >> %TMPFILE1%
ECHO 	    EXIT WHEN csr_objects%%NOTFOUND; >> %TMPFILE1%
ECHO 	    BEGIN >> %TMPFILE1%
ECHO 	      EXECUTE IMMEDIATE 'ALTER PROCEDURE ' ^|^| sName ^|^| ' COMPILE'; >> %TMPFILE1%
ECHO 	    EXCEPTION >> %TMPFILE1%
ECHO 	      WHEN OTHERS THEN NULL;>> %TMPFILE1%
ECHO 	    END; >> %TMPFILE1%
ECHO 	  END LOOP; >> %TMPFILE1%
ECHO 	  CLOSE csr_objects; >> %TMPFILE1%
ECHO 	END; >> %TMPFILE1%
ECHO END; >> %TMPFILE1%
ECHO / >> %TMPFILE1%
ECHO EXIT 0; 						>> %TMPFILE1%
SQLPLUS %CONN_STRING% @%TMPFILE1% 			>> %LOGFILE1%
IF NOT %ERRORLEVEL% == 0 GOTO ErrorRecompile
ECHO Recompilation finished 

GOTO EndSuccess

:ErrorDropUsers
ECHO ERROR: Can not drop database users that are currently connected! >> %LOGFILE1%
ECHO ERROR: Can not drop database users that are currently connected!
GOTO EndError

:ErrorCreateUsers
TYPE %LOGFILE1%
ECHO ERROR: Can not create database users required for Converge! >> %LOGFILE1%
ECHO ERROR: Can not create database users required for Converge!
GOTO EndError

:ErrorImportFile
ECHO ERROR: Can not find import file %DUMP_FILE% >> %LOGFILE1%
ECHO ERROR: Can not find import file %DUMP_FILE%
GOTO EndError

:ErrorComputerNames
TYPE %LOGFILE1%
ECHO ERROR: Unable to set computer names! >> %LOGFILE1%
ECHO ERROR: Unable to set computer names!
GOTO EndError

:ErrorDaUser
TYPE %LOGFILE1%
ECHO ERROR: Unable to add user %USERNAME% for DA GUI!  >> %LOGFILE1%
ECHO ERROR: Unable to add user %USERNAME% for DA GUI!
GOTO EndError

:ErrorNvarchar
TYPE %LOGFILE1%
ECHO ERROR: Unable to convert required fields to nvarchar!  >> %LOGFILE1%
ECHO ERROR: Unable to convert required fields to nvarchar!
GOTO EndError

:ErrorAEAlarmData
TYPE %LOGFILE1%
ECHO ERROR: Unable to add foreign key for ae alarm data!  >> %LOGFILE1%
ECHO ERROR: Unable to add foreign key for ae alarm data!
GOTO EndError

:ErrorRecompile
TYPE %LOGFILE1%
ECHO ERROR: Unable to recompile!  >> %LOGFILE1%
ECHO ERROR: Unable to recompile!
GOTO EndError

:Usage
ECHO ** You may set optional environment variables if required
ECHO ** different from their default value:
ECHO **   SYSTEM_PWD            default: %SYSTEM_PWD%
ECHO **   NET_SERVICE           default: ATHENADB
ECHO **   DUMP_FILE             default: %DUMP_FILE%
ECHO **   DA_SERVER             default: %COMPUTERNAME%
ECHO **   DP_SERVER             default: %COMPUTERNAME%
ECHO **
GOTO EndError

:EndError
IF EXIST %TMPFILE1% DEL %TMPFILE1%
REM IF EXIST %LOGFILE1% DEL %LOGFILE1%
ECHO Import Script finished >> %LOGFILE1%
ECHO Import Script finished
IF "%BATCH%" == "YES" EXIT 1
GOTO EndBatch

:EndSuccess
IF EXIST %TMPFILE1% DEL %TMPFILE1%
REM IF EXIST %LOGFILE1% DEL %LOGFILE1%
ECHO Import Script finished >> %LOGFILE1%
ECHO Import Script finished
IF "%BATCH%" == "YES" EXIT 0
:EndBatch