GO
/****** Object:  StoredProcedure [dbo].[JIRA_Project_Auto_Archiver] ******/
SET ANSI_NULLS ON
GO
SET QUOTED_IDENTIFIER ON
GO
-- =============================================
-- Author:		Dalton, Kevin
-- Create date: <Create Date,,>
-- Description:	Automation to archive projects based on permission scheme
-- =============================================
ALTER PROCEDURE [dbo].[JIRA_Project_Auto_Archiver]  
	-- Add the parameters for the stored procedure here
AS
BEGIN
------------------------ONLY MODIFY THESE
	
	DECLARE @baseURL NVARCHAR(255);
	SET @baseURL = ('https://jira.yoururl.com'); -- TARGET INSTANCE TO UPDATE

	-------------------------
	DECLARE @authHeader NVARCHAR(64);
	DECLARE @contentType NVARCHAR(64);
	DECLARE @postData NVARCHAR(2000);

	DECLARE @jiraIssuePkey  NVARCHAR(2000)

	DECLARE @responseText NVARCHAR(2000);
	DECLARE @responseXML NVARCHAR(2000);
	DECLARE @ret INT;
	DECLARE @status NVARCHAR(32);
	DECLARE @statusText NVARCHAR(32);
	DECLARE @token INT;
	DECLARE @url NVARCHAR(256);
	DECLARE @data_table VARCHAR(50);
	DECLARE @PUT NVARCHAR(20);
	-- SET NOCOUNT ON added to prevent extra result sets from
	-- interfering with SELECT statements.
	SET NOCOUNT ON;

	SET @authHeader =  N'Basic TOKEN';
	SET @contentType = N'application/json';
	SET @PUT = 'PUT';

		IF CURSOR_STATUS('global', 'solution_cursor') >= -1
		BEGIN
		  CLOSE solution_cursor
		  DEALLOCATE solution_cursor
		END

	BEGIN TRY
DECLARE solution_cursor CURSOR fast_forward FOR SELECT p.pkey
--p.pname, WF.NAME
FROM dbo.PROJECT P
JOIN dbo.NODEASSOCIATION NA
	ON P.ID = NA.SOURCE_NODE_ID
JOIN dbo.permissionscheme PS
	ON NA.SINK_NODE_ID = PS.ID
where NA.SINK_NODE_ENTITY = 'PermissionScheme' and PS.NAME = 'Read Only Permission Scheme'
and p.id not in (select p.id
from dbo.propertyentry pe 
	join dbo.project p 
		on pe.entity_id=p.id 
where property_key = 'jira.archiving.projects')
and p.id not in (select pro.id from dbo.project pro
where pro.pkey in ('EXCLUDED PROJECT KEYS')
)



			
		FOR READ ONLY;

		OPEN solution_cursor
		FETCH NEXT FROM solution_cursor INTO
		@jiraIssuePkey;

	WHILE @@fetch_status = 0
		BEGIN

				BEGIN
		SET @url = @baseURL + '/rest/api/2/project/' + @jiraIssuePkey + '/archive'
		PRINT '';
		PRINT @url;
	
			EXEC @ret = sp_OACreate N'MSXML2.ServerXMLHTTP.3.0'
							   ,@token OUT;
            --print 'sp_OACreate MSXML2.ServerXMLHTTP.3.0: '
			IF @ret <> 0
				RAISERROR ('Unable to open HTTP connection.', 10, 1);

			-- Send the request.
			EXEC @ret = sp_OAMethod @token
							   ,'open'
							   ,NULL
							   ,@PUT
							   ,@url
							   ,'false';
			--print 'sp_OAMethod open: '
			EXEC @ret = sp_OAMethod @token
							   ,'setRequestHeader'
							   ,NULL
							   ,'Authorization'
							   ,@authHeader;
			--print 'sp_OAMethod setRequestHeader Authorization: ' 
			EXEC @ret = sp_OAMethod @token
							   ,'setRequestHeader'
							   ,NULL
							   ,'Content-Type'
							   ,@contentType;
			--print 'sp_OAMethod setRequestHeader content-type: ' 
			EXEC @ret = sp_OAMethod @token
							   ,'send'
							   ,NULL;

			-- Handle the response.
			--print 'sp_OAMethod send: ' 
			EXEC @ret = sp_OAGetProperty @token
									,'status'
									,@status OUT;
			--print 'sp_OAMethod status: ' 
			EXEC @ret = sp_OAGetProperty @token
									,'statusText'
									,@statusText OUT;
			--print 'sp_OAMethod statustext: ' 
			EXEC @ret = sp_OAGetProperty @token
									,'responseText'
									,@responseText OUT;
			--print 'sp_OAMethod responsetext: ' 

			-- Show the response.
			PRINT N'Status: ' + @status + ' (' + @statusText + ')';
			--PRINT N'Response text: ' + @responseText;
			
			-- Close the connection.
			EXEC @ret = sp_OADestroy @token;
			IF @ret <> 0
				RAISERROR ('Unable to close HTTP connection.', 10, 1);
		END

		FETCH NEXT FROM solution_cursor INTO
		@jiraIssuePkey;
		END
		CLOSE solution_cursor;
		DEALLOCATE solution_cursor;
	END TRY
	
	
	BEGIN CATCH
		DECLARE @ErrorMessage VARCHAR(8000) = ERROR_MESSAGE() + ' at line # ' + convert(VARCHAR(255), ERROR_LINE())
		,@ErrorSeverity INT = ERROR_SEVERITY()
		,@ErrorState INT = ERROR_STATE()

		RAISERROR (@ErrorMessage, @ErrorSeverity, @ErrorState);
	END CATCH

END
