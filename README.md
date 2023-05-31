# APEX-Face-Identification

# Getting started
For the API calls, we need a KEY that we can get on the Azure Portal. When Logged to the portal, open the left menu and click on “Create a resource”, search for “Face” from the AI + Machine Leaning Category

# Installation
Install apex-face-identification-demo.sql

Add the API-Key to occurences of {API-KEY-HERE}

**Getting started**

For the API calls, we need a KEY that we can get on the Azure Portal. When Logged to the portal, open the left menu and click on “Create a resource”, search for “Face” from the AI + Machine Leaning Category. The free tier allows us to perform 20 calls per minute and 30K Calls per month, which is more than enough to develop and even to use on some apps.

**Create a Faces Database**
Microsoft Azure Face API Create a Faces Database

There are different ways to work with the Face API. For this demo app I am doing the following.

Create a person group called users
Create a person on this group called Rodrigo Mesquita and we also save the app user id (‘RODRIGOM’)
Add the person’s face using an image blob stored in a database table.

    DECLARE 
    l_clob CLOB;
    l_body CLOB;
    lv_group_id VARCHAR2(100) := 'users';
    lv_person_id VARCHAR2(100);
    l_image BLOB;
    BEGIN apex_web_service.g_request_headers(1).name := 'Ocp-Apim-Subscription-Key';
    apex_web_service.g_request_headers(1).value := '<add the API KEY here>';
    apex_web_service.g_request_headers(2).name := 'Content-Type';
    apex_web_service.g_request_headers(2).value := 'application/json';


Create a Person Group: Create a new person group passing JSON with specified personGroupId, name, user-provided userData and recognitionModel.

    APEX_JSON.initialize_clob_output;
    APEX_JSON.open_object;
    APEX_JSON.write('name', lv_group_id);
    APEX_JSON.write(
      'userData', 'user-provided data attached to the person group.'
    );
    APEX_JSON.write(
      'recognitionModel', 'recognition_03'
    );
    APEX_JSON.close_object;
    l_body := APEX_JSON.get_clob_output;
    APEX_JSON.free_output;
    l_clob := apex_web_service.make_rest_request(
      p_url => 'https://northeurope.api.cognitive.microsoft.com/face/v1.0/persongroups/' || lv_group_id, 
      p_http_method => 'PUT', p_body => l_body
    );

**Create a Person in the group**

    APEX_JSON.initialize_clob_output;
    APEX_JSON.open_object;
    APEX_JSON.write('name', 'Rodrigo');
    APEX_JSON.write('userData', 'RODRIGOM');
    APEX_JSON.close_object;
    l_body := APEX_JSON.get_clob_output;
    APEX_JSON.free_output;
    l_clob := apex_web_service.make_rest_request(
      p_url => 'https://northeurope.api.cognitive.microsoft.com/face/v1.0/persongroups/' || lv_group_id || '/persons', 
      p_http_method => 'POST', p_body => l_body
    );
    APEX_JSON.parse(l_clob);
    lv_person_id := APEX_JSON.get_varchar2(p_path => 'personId');

**Add a face to the person: Each person entry can hold up to 248 faces.**

    -- get user image 
    SELECT 
      IMAGE INTO l_image 
    FROM 
      USER_IMAGES 
    WHERE 
      USER_ID = 'RODRIGOM';
    apex_web_service.g_request_headers.delete(2);
    apex_web_service.g_request_headers(2).name := 'Content-Type';
    apex_web_service.g_request_headers(2).value := 'application/octet-stream';
    l_clob := apex_web_service.make_rest_request(
      p_url => 'https://northeurope.api.cognitive.microsoft.com/face/v1.0/persongroups/' || lv_group_id || '/persons/' || lv_person_id || '/persistedFaces?detectionModel=detection_01', 
      p_http_method => 'POST', p_body_blob => l_image
    );

Train the Group: This call submits a person group training task. Training is a crucial step as only a trained person group can be used. Each time we create or change a group, a face or a person from a group, then that group should be trained again

    apex_web_service.g_request_headers.delete(2);
    l_clob := apex_web_service.make_rest_request(
      p_url => 'https://northeurope.api.cognitive.microsoft.com/face/v1.0/persongroups/' || lv_group_id || '/train', 
      p_http_method => 'POST', p_body => '{body}'
    );
    END;

If no errors occurred, we have a group called users, with a person called Rodrigo Mesquita, a face for this person and this group is trained and ready. To check the training status we can call PersonGroup – Get Training Status but that is not necessary here as we have just one face so the training should only take seconds.




