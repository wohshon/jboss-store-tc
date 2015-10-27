# jboss-store
## EAP 6 and MySQL demo deployable on OpenShift v3 via S2i build

A shopping cart demo that showcase EAP 6 and MySQL images on OpenShift v3.  
Shopping cart object is stored in http session  
Upon checkout, orders will be persisted in MySQL database and can be viewed by the user in session   
Database schema is generated by JPA  

### 1. Pre-requisites
- Clone the following repo to your local environment  
  https://github.com/jboss-openshift/application-templates  
  In this example, it is assumed the the repo is cloned to the current directory.  

- This demo uses official red hat images so appropriate subscriptions maybe required  

Advisable to update the imagestreams and templates to the latest ones even if you already have them in your existing environment:

#### 1.1 Create JBoss imagestreams  
  As a **system:admin** 
  ```
  oc create -f application-templates/jboss-image-streams.json -n openshift
  ```
  
#### 1.2 Create eap6-mysql-sti template
  Open up the file application-templates/templates/eap/eap6-mysql-sti.json, ensure the "type" attribute
  for github are all in small caps
  (The original value is 'Github' which will cause an error during deployment) 
  
  The correct value should look like:
  ```
                  "triggers": [
                      {
                          "type": "github",
                          "github": {
                              "secret": "${GITHUB_TRIGGER_SECRET}"
                          }
                      },
  ```
  
  As a **system:admin** 
  ```
  oc create -f application-templates/templates/eap/eap6-mysql-sti.json -n openshift
  ```

### 2. Creating the Project
- As a **normal user**, 
```
oc new-project openshift-store 
```
(Or create a project via the web console)

- **important step:** Create the required service accounts via the CLI (remember to create this under the right project)
```
oc create -f application-templates/secrets/eap-app-secret.json
```

#### 2.1 Creating the Application

##### Create the application via the Web Console using S2i  

- Select the project you have just created
- Click 'Create' at the top right hand side
- On the next page, click on Browse all templates at the left bottom
- Click on eap6-mysql-sti (or whatever name given to the eap and mysql template you have created)
- Click "Select template" on the pop up to proceed
- On the next page, click on Edit Parameter:
- The important parameters to take note of are as follows:
  - **GIT_URI** : https://github.com/wohshon/jboss-store
  - **GIT_REF** : master
  - **GIT_CONTEXT_DIR**: &lt;leave it blank&gt;
  - **DB_JNDI**: java:jboss/datasources/storeDS  
  The rest of the paramters can be arbitrary or auto generated
 
- Click on Create at the bottom of the page and wait for the build to complete
 
 Once the build is completed, do wait for 1-2 minutes as the EAP pod is still in the midst of starting up.  
 You can view the startup logs in the EAP pod by
```
oc logs -f <eap pod name>
```
Once the eap instance is up, access the deom at 
http://&lt;app-name&gt;.&lt;your domain&gt;

### 3. Others
 HTTPS endpoint does not work in this version of the demo.
 



