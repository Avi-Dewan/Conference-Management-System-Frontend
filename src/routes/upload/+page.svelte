<script>

import { createClient } from '@supabase/supabase-js'
import { v4 as uuidv4 } from 'uuid';


const supabase = createClient(
  process.env.REACT_APP_SUPABASE_URL,
  process.env.REACT_APP_ANON_KEY,
)

function App() {

  const [userId, setUserId] = useState('');
  const [media, setMedia] = useState([]);




  async function uploadImage(e) {
    let file = e.target.files[0];


    const { data, error } = await supabase
      .storage
      .from('uploads')
      .upload(userId + "/" + uuidv4(), file)


  }


}







  useEffect(() => {
    getUser();
    getMedia();
  }, [userId])


  </script>
  
  <style>
    /* Add your styles here */
  </style>
  
  <div>
    <input type="file" on:change={uploadImage} />
    <button on:click={uploadImage}>Upload File to Supabase Storage</button>
  </div>
  
  