<script>

  import { onMount } from 'svelte';
  
  let data = false
  let exclude = ['StreamOrder', 'ID', 'Format_Level', 'Format_Settings_CABAC', 'Format_Settings_RefFrames', 'Encoded_Library_Settings', 'extra']
  
  onMount(async () => {
    
    const fileinput = document.getElementById('fileinput')
    const output = document.getElementById('output')
    
  
  const onChangeFile = (mediainfo) => {
    const file = fileinput.files[0]
    if (file) {
      // output.value = 'Working…'
  
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
           data = JSON.parse(result)
           
           
          
       console.log(data.media.track[1])
          // output.innerHTML = result
        })
        .catch((error) => {
          console.log(`An error occured:\n${error.stack}`)
        })
    }
  }
  
  MediaInfo({ format: 'JSON' }, (mediainfo) => {
    fileinput.addEventListener('change', () => onChangeFile(mediainfo))
  })
  
  });
  
  function selectFile(){
    document.querySelector("#fileinput").click();
  }
</script>


<a href="https://www.instantfilm.net"
  ><img
    src="./img/logo.png"
    id="logo"
/></a>
<div class="wrap">
  <main id="main">
    <div class="wdgt wdgt-main">
      <div class="wdgt-content p-3">
        
        
        <button on:click={selectFile}>Select File</button>
        
 <input type="file" id="fileinput" name="fileinput" style="display: none;" />
 
 {#if data}
 {#each data.media.track as item}
 
 {#if item['@type'] !== 'General'}
 <h4 class="mt-4">{item['@type']}</h4>
 
 <table class="table table-striped">
   <tbody>

     {#each Object.keys(item) as key}
     
     {#if !exclude.includes(key)}
     <tr>
       <th scope="row" class="w-50">{key.replaceAll('_', ' ')}</th>
       <td class="text-end"><span style="display: inline-block; max-width: 140px; overflow: hidden;">{item[key]}</span></td>
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