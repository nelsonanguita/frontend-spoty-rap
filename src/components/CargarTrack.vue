<template>
    <div>
  
        <form action="https://backend-spotyrap-production.up.railway.app/tracks" enctype="multipart/form-data" method="POST">
            <input type="file" name="tracks" multiple >
            <input type="submit" value="Upload track">
        </form>
    </div>
  </template>
  
  <script>
  
  export default {
      name: "CargaTrack",
      data(){
        return{
          tracks:[],
        }
      },
      methods:{
          sendToServer : async function (formData) {
          const data = await fetch('http://localhost:3000/avatar', {
            method: 'POST',
            // mode: 'cors',
            // cache: 'no-cache',
            // credentials: 'same-origin',
            headers: {
              'Connection': 'keep-alive',
              'Accept': '*/*',
              'Content-Type': 'multipart/form-data; boundary=helldfkljksf',
            },
            body: formData,
          });
  
          console.log(data.json());
        }
      }
  
  }
  </script>
  
  <style>
  
  </style>