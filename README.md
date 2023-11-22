### Required Imports 
- First install axios library
```JavaScript
import React, { useState } from 'react'
import axios from 'axios'
import Cytomine  from '../helpers/cytomine';
import ImageInstanceCollection from '../helpers/image-instance-collection';
import constants from '../helpers/constants';

```


### State setup

```JavaScript
const [image, setImage]=useState([])
const [uploadprogress, setUploadProgress]=useState(0)
let cytomine =  new Cytomine('http://core-cytomine.roycelabs.xyz'); 
// cytomine is the location of Cytomine core

```


### HTML form setup

```
<!-- Render added file -->

{renderFileList()}

<form onSubmit={(e)=>uploadImage(e)}>
<input onChange={(e)=>handleImageUpload(e.target.files[0])} className='hidden'  id="actual-btn"  type='file' />
</form>

<label for="actual-btn" >Add File</label>

<button
  type="button"
  onClick={()=>uploadAll()}
  >
  Upload All
</button>

<divstyle={{width: `${uploadprogress}%`}}>
    {uploadprogress}%
  </div>


```






### Javascript functions

- 

```Javascript
// Render Added files in the UI

const renderFileList = () => (<>
     {[...image].map((f, i) => (
        <>
        <tr>
          <td>
          {f.name} - {f.type}
            
          </td>
          <td>
          <button onClick={()=>uploadIndividually(f)} >Upload</button>

          </td>
        </tr>
          
          
        </>
      ))}
    </>
    

const uploadIndividually=async(image)=>{
  let my_signature= await getSignature();
  uploadImage(image,my_signature)

 }

  const uploadAll=async()=>{
    
    [...image].forEach(image=>{
      console.log(image)
     

    })

  }

    // Handling image upload
    const handleImageUpload=(file)=>{
      
      setUploadProgress(0)
      setImage(image=>[...image,file])
    

    }

   
    // Initilaize a variable to hold date
    let mydate=''

    // Upload URI in cytomine
    function uriVue() {
      return '/upload';
    }
    // Constructing query string to be sent
    function queryString() {
      let str = `cytomine=${constants.CYTOMINE_CORE_HOST}`;
      if(1==1) {
        str += `&idStorage=`+'51';
      }
      if(1==1) {
        str += `&idProject=520`;
      }
      return str;
    }

    // Creating a signature

    async function getSignature(){
      mydate=new Date().toISOString()

      console.log("Date:"+mydate)
      console.log("uri:"+uriVue())
      console.log("query String:"+queryString())

      let signature = await Cytomine.instance.fetchSignature({
        uri: uriVue(),
        queryString: queryString(),
        method: 'POST',
        date: mydate
      });
      // alert(signature)


      return await signature;
    }

```


- FIle upload function 
```javascript

const uploadImage=async (image,my_signature)=>{
     
      let formData = new FormData();
      formData.append('files[]', image);
     
      let publicKey="113b9814-dd48-4113-8f3f-9a58c96b6016"
      
    //    
      axios.post(
        constants.CYTOMINE_UPLOAD_HOST +  uriVue() + '?' +  queryString(),
        formData,
        {
          headers: {
            'authorization': `CYTOMINE ${publicKey}:${my_signature}`,
            'dateFull': mydate, // will replace actual date value, so that signature is valid
            'content-type-full': 'null' // will erase actual content-type value, so that signature is valid
          },
          transformRequest: [function (data, headers) {
            headers['Content-Type'] = 'multipart/form-data'
             
              headers['authorization'] = `CYTOMINE ${publicKey}:${my_signature}`
              headers['dateFull'] = mydate
              headers['content-type-full'] = 'null'
              
              
            return (data)
          }],
          onUploadProgress: progress => {
            console.log(Math.floor((progress.loaded * 100) / progress.total))
            setUploadProgress(Math.floor((progress.loaded * 100) / progress.total))
            
          },
          cancelToken: axios.CancelToken.token

        }
      ).then(response =>  {
        console.log("This is response after uploading")
        console.log(response.data[0].uploadedFile.originalFilename)
        // Fetch File so as to construct the path

         handleFetchFile(response.data[0].uploadedFile.originalFilename)
       

        return ;

      }).catch(error => {
        if(!axios.isCancel(error)) {
          console.log(error);
          // fileWrapper.uploadedFile = false;
        }
      }).finally(() => {
        
        console.log("Finally")
      });

    }


```

### Constructing the file path

```javascript
const handleFetchFile=async (name)=>{
   
    let collection1 = new ImageInstanceCollection({filterKey: 'project', filterValue: 520});
   let abcd=await collection1.fetchAll()

   let my_data=abcd._data

  //  console.log(my_data[0])

   const result = my_data.filter((data) => data.originalFilename
   === name);

 
   let project=520
  let full_url=`/project/${project}/image/${result[0].id}`

//   Call the function to store path in the backend

// ful_ur: complete URL
// result[0].id: Image ID
// name: is the name of the uploaded image
  saveToFirebase(full_url,result[0].id,name)


    

    return ;
    

}

```
