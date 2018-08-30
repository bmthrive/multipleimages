<?php

if(isset($_POST['submit'])){
    
    // Include the database configuration file
    
    include_once 'dbConfig.php';
    
    // File upload configuration
    
    $targetDir = "uploads/";
    $allowTypes = array('jpg','png','jpeg','gif','.pdf','.doc','.csv','.xlsx','.xlsm' , '.xls', '.txt' , '.htm', '');
    
    $statusMsg = $errorMsg = $insertValuesSQL = $errorUpload = $errorUploadType = '';

    $i = 0; 
    
    if(trim(array_filter($_FILES['files']['name'])) == false ){

        foreach($_FILES['files']['name'] as $key=>$val){

            $i = $i + 1; 

            $ext = pathinfo($_FILES['files']['name'][$key], PATHINFO_EXTENSION);

            // File upload path
            $fileName = basename($_FILES['files']['name'][$key]);
            
            // Check whether file type is valid
            $fileType = pathinfo($targetFilePath,PATHINFO_EXTENSION);
            
            if(in_array($fileType, $allowTypes)){

                $now = new DateTime();

                $fileName1 = $now->getTimestamp().$i.'.'.$ext;

                $targetFile = $targetDir.$fileName1;


                // Upload file to server
                if(move_uploaded_file($_FILES["files"]["tmp_name"][$key], $targetFile )){
                    
                    // Image db insert sql
                    $insertValuesSQL .= "('".$fileName."', NOW()),";
                    
                }else{
                    
                    $errorUpload .= $_FILES['files']['name'][$key].', ';
                }
            }
            else{
                $errorUploadType .= $_FILES['files']['name'][$key].', ';
            }
        }
        
        if(!empty($insertValuesSQL)){

            $insertValuesSQL = trim($insertValuesSQL,',');

            // Insert image file name into database
            $insert = $db->query("INSERT INTO images (file_name, uploaded_on) VALUES $insertValuesSQL");

            if($insert){

                $errorUpload = !empty($errorUpload)?'Upload Error: '.$errorUpload:'';
                $errorUploadType = !empty($errorUploadType)?'File Type Error: '.$errorUploadType:'';
                $errorMsg = !empty($errorUpload)?'<br/>'.$errorUpload.'<br/>'.$errorUploadType:'<br/>'.$errorUploadType;
                $statusMsg = "Files are uploaded successfully.".$errorMsg;

            }else{
                $statusMsg = "Sorry, there was an error uploading your file.";
            }
        }
    }else{

        $statusMsg = 'Please select a file to upload.';

    }
    
    // Display status message
    echo $statusMsg;
}
?>

<form action="" method="post" enctype="multipart/form-data">
    Select Image Files to Upload:
    <input type="file" name="files[]" multiple >
    <input type="submit" name="submit" value="UPLOAD">
</form>

