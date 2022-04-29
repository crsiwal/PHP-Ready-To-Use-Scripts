
## Upload File to cloud using Codeigniter 3

The following function shows how to upload the file to cloud in Codeigniter 3 PHP:

```php
<?php
function uploadFile($filename, $config) {
    if ($filename != "") {
        $ci = &get_instance();
        try {
            if (!is_dir($config['upload_path'])) {
                mkdir($config['upload_path'], 0777, TRUE);
            }
            $ci->load->library('upload', $config);
            $uploadedFile = $ci->upload->do_upload($filename);
            if ($uploadedFile === TRUE) {
                return $ci->upload->data();
            }
        } catch (Exception $exc) {
            $ci->logger->error($exc->getTraceAsString());
            return FALSE;
        }
    }
    return FALSE;
}
?>
```
