# mongodb-file-api
Create Spring Boot API that allows file uploading and serving files.


## Part 1: Implement File uploader API

1. Change the Database configuration with your own credentials in the following files:
        
    a. *application.properties*:
    ```text
        #spring.data.mongodb.uri=mongodb://localhost/santiago
        spring.data.mongodb.uri=mongodb+srv://<username>:<password>@cluster0-dzkk5.mongodb.net/test?retryWrites=true&w=majority
    ```
    b. *AppConiguration.java*
    ```Java
        MongoClientURI uri = new MongoClientURI(
            "mongodb+srv://<username>:<password>@cluster0-dzkk5.mongodb.net/test?retryWrites=true&w=majority");

        MongoClient mongoClient = new MongoClient(uri);

        return new SimpleMongoDbFactory( mongoClient, "<database_name>");
    ```
2. Go to the *Application* class run the project.

3. Go to your database and verify that the file was created under a fs.files and fs.chunks documents. 

4. Go to the RESTController class and inject the *GridFsTemplate* bean by using the *@Autowired* annotation.

5. Implement the *getFileByName* method:

    * Find the file with the following code:
        ````Java
          GridFSFile file = gridFsTemplate.findOne(new Query().addCriteria(Criteria.where("filename").is(filename)));
        ````
    * If the file does not exist return the Not Found HTTP error:
    
        ````Java
            return new ResponseEntity<>(HttpStatus.NOT_FOUND);
        ````
    * If the file exist return the correct resource:
        ````Java
                GridFsResource resource = gridFsTemplate.getResource(file.getFilename());
                return ResponseEntity.ok()
                    .contentType(MediaType.valueOf(resource.getContentType()))
                    .body(new InputStreamResource(resource.getInputStream()));
        ````    
    
5. Verify that the *getFileByName* endpoint works and it does returns the *lion.jpeg* created on 1.


6. Implement the *handleFileUpload* method. Make sure you return the static url to access the uploaded file by using the *getFileByName* endpoint.

    ````Java
       //Stores the file into MongoDB
        gridFsTemplate.store(file.getInputStream(), fileName, file.getContentType());
    ````

## Part 2: Integrate file uploader with React JS project

1. Add the following input to the Todo form:

    ````Javascript
        <input type="file" id="file" onChange={this.handleInputChange}/>
    ````
    
2. Implement and bind the *handleInputChange* method:

    ````Javascript
         handleInputChange(e) {
                this.setState({
                    file: e.target.files[0]
                });                
            }
    ````

3. Change the *handleSubmit* method so it uploads the file first:

    ````Javascript
    
            let data = new FormData();
            data.append('file', this.state.file);
    
            this.axios.post('files', data)
                .then(function (response) {
                    console.log("file uploaded!", data);
            })
            .catch(function (error) {
                console.log("failed file upload", error);
            });
    
    ```` 
    
4. Run your React project and verify that the file uploader works.


## Part 3: Upgrade your Todo to accept files

1. Inject the *TodoRepository* into the *RESTController using *@Autowired* annotation.

2. Implement the *createTodo* and *getTodoList* methods of the *RESTController*.

3. Modify the *handleSubmit* method so it does call the the API to create the *Todo* entry on the server and database:

    * Make asynchronous calls to upload file to the server
    * Once the file upload promise is fulfill, then call then save the Todo entry to the API using the *POST* method.
    * Remember to save the *this* context into a variable to use it into the nested scopes!
   
4. Modify your *Todo.js* on your React project so the it display the Todo image:

    ````Javascript
          <td>{this.props.fileUrl ? <img src={this.props.fileUrl} /> : <div/>}</td>
    ```` 

5. If you have not done it implement and call the method to load the Todo list from the server:

    ```javascript
         loadDataFromServer() {
        
                let that = this;
        
                this.axios.get("todo").then(function (response) {
                    console.log("This is my todolist:  ", response.data);
                    that.setState({items: response.data})
                })
                    .catch(function (error) {
                        console.log(error);
                    });
            }

    ```    
6. Implement the logic that if the file is not an image but a pdf file, then a file icon appears on the Todo list that when is clicked then it redirects to the download page.
    
7. Verify that the project works as expected.

    
    
