<script>

  import { onMount } from 'svelte';
  
  let data = false
  let exclude = ['StreamOrder', 'ID', 'Format_Level', 'Format_Settings_CABAC', 'Format_Settings_RefFrames', 'Encoded_Library_Settings', 'extra']
  
  let loading = false;
  let error = false;
  
  onMount(async () => {
    
    const fileinput = document.getElementById('fileinput')
    const output = document.getElementById('output')
    
  
  const onChangeFile = (mediainfo) => {
    const file = fileinput.files[0]
    if (file) {
      loading = true;
      data = false;
      error = false;
  
      const getSize = () => file.size
  
      const readChunk = (chunkSize, offset) =>
        new Promise((resolve, reject) => {
          const reader = new FileReader()
          reader.onload = (event) => {
            if (event.target.error) {
              reject(event.target.error)
            }
            resolve(new Uint8Array(event.target.result))
          }
          reader.readAsArrayBuffer(file.slice(offset, offset + chunkSize))
        })
  
      mediainfo
        .analyzeData(getSize, readChunk)
        .then((result) => {
          
          let myresult = JSON.parse(result);
          
          if(myresult.media.track[1]){
            data = myresult
             loading = false;
          }else{
            error = "Unknown video type"
            loading = false;
          }
           
       
          
        })
        .catch((er) => {
          console.log(`An error occured:\n${error.stack}`)
          error = "An error has occurred";
          loading = false;
        })
    }
  }
  
  MediaInfo({ format: 'JSON', locateFile: (path, prefix) => prefix + path}, (mediainfo) => {
    fileinput.addEventListener('change', () => onChangeFile(mediainfo))
  })
 
  });
  
  function selectFile(){
    document.querySelector("#fileinput").click();
  }
</script>

<!--
<a href="./../"
  ><img
    src="https://www.cinetools.net/img/cinetools.png"
    id="logo"
/></a>
-->

<div class="text-center"><h5><a href="https://www.clipflare.com"><img src="img/cinetools.png" id="logo" /></a></h5></div>

<div class="wrap">
  <main id="main">
    <div class="wdgt wdgt-main">
      <div class="wdgt-content p-3">
        
        <div class="alert alert-warning text-center mb-3">
           
           Get technical info about your video file, like codec, bitrate, etc.
           
         </div>

        
        
        <button on:click={selectFile} class="btn btn-dark w-50 d-block m-auto mb-3">Select Video File</button>
        
 <input type="file" id="fileinput" name="fileinput" style="display: none;" />
 

 
 

 {#if loading}
 <img src="./img/loading.png" class="rotate loading" />
 {/if}
 
 {#if error}
 <div class="mt-3">
 {error}
 </div>
 {/if}

 
 {#if data}
 {#each data.media.track as item}
 
 {#if item['@type'] !== 'General'}
 <h4 class="mt-4">{item['@type']}</h4>
 
 <table class="table table-striped">
   <tbody>

     {#each Object.keys(item) as key}
     
     {#if !exclude.includes(key)}
     <tr>
       <th scope="row" class="w-50 text-capitalize">{key.replaceAll('_', ' ')}</th>
       <td class="text-truncate">{item[key]}</td>
     </tr>
     {/if}
     {/each}
    
   </tbody>
 </table>
 
 {/if}
 {/each}
 {/if}

      </div>
    </div>
  </main>
</div>