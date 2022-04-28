
## Common Template and Layout Library for Codeigniter 3

The following class will be used for show the all pages from Controller in Codeigniter 3 PHP:

Uses of this Library Class
```php
<?php

defined('BASEPATH') or exit('No direct script access allowed');

/* * ******************************************************************
 *  Author: Rahul Siwal
 *  Website: https://www.rsiwal.com 
 *  All Rights Reserved.
 */
 
class ExampleController extends CI_Controller {

    public function __construct() {
        parent::__construct();
        $this->load->library('Template');
    }

    public function index() {
        $data = array(
            "popup" => array('login'),
        );
        $this->template->addMeta('title', 'Title of the page will be configure here');
        $this->template->addMeta('description', 'Set the description of page.');
        $this->template->show('pages/about', $data, 'page_name');
    }

}

```

For using above Save code you should add the following code in Library directory with name Template.php

```php
<?php

defined('BASEPATH') OR exit('No direct script access allowed');

/* * ******************************************************************
 *  Author: Rahul Siwal
 *  Website: https://www.rsiwal.com 
 *  All Rights Reserved.
 */
 
if (!class_exists('Template')) {

    class Template {

        private $head;
        private $js;
        private $header;
        private $footer;
        private $left_sidebar;
        private $right_sidebar;

        /**
         * 
         */
        public function __construct() {
            $this->ci = & get_instance();
            $this->init();
        }

        /**
         * 
         * @param type $template
         * @param type $data
         */
        public function show($template, $data = array(), $page_name = 'unknown') {
            $this->ci->logger->start();
            $this->setPageName($page_name);
            if ($this->ci->user->isLogin() === TRUE) {
                $data['menulist'] = $this->ci->dashboard->getMenuList();
                $data['globaldata'] = $this->ci->user->getUserActionData();
            }
            $footer_end = array(
                'js' => $this->js,
                'popup' => $this->prepairPopupLocation((isset($data['popup']) && is_array($data['popup'])) ? $data['popup'] : array())
            );
            $sanitize = (ENVIRONMENT == 'development' || in_array($page_name, ["xyzpage"])) ? FALSE : TRUE;
            $this->head['side_bar_open'] = $data['side_bar_open'];
            $this->head['sidebar_left'] = $this->left_sidebar;
            $this->head['header'] = $this->header;
            switch ($sanitize) {
                case TRUE:
                    $content = '';
                    $content .= $this->ci->load->view('header/header_db_head', $this->head, TRUE);
                    $content .= $this->ci->load->view('header/' . $this->header, $data, TRUE);
                    $content .= ($this->left_sidebar != FALSE) ? $this->ci->load->view('sidebar/' . $this->left_sidebar, $data, TRUE) : '';
                    $content .= $this->ci->load->view($template, $data, TRUE);
                    $content .= ($this->right_sidebar != FALSE) ? $this->ci->load->view('sidebar/' . $this->right_sidebar, $data, TRUE) : '';
                    $content .= $this->ci->load->view('footer/' . $this->footer, $data, TRUE);
                    $content .= $this->ci->load->view('footer/footer_db_end', $footer_end, TRUE);
                    $this->sanitize($content);
                    break;
                case FALSE:
                    $this->ci->load->view('header/header_db_head', $this->head);
                    $this->ci->load->view('header/' . $this->header, $data);
                    ($this->left_sidebar != FALSE) ? $this->ci->load->view('sidebar/' . $this->left_sidebar, $data) : '';
                    $this->ci->load->view($template, $data);
                    ($this->right_sidebar != FALSE) ? $this->ci->load->view('sidebar/' . $this->right_sidebar, $data) : '';
                    $this->ci->load->view('footer/' . $this->footer, $data);
                    $this->ci->load->view('footer/footer_db_end', $footer_end);
                    break;
            }
            $this->ci->logger->end("Template:show");
        }

        /**
         * 
         * @param type $key
         * @param type $value
         */
        public function addMeta($key = '', $value = '') {
            if ($key != "") {
                switch ($key) {
                    case 'title':
                        $this->head['title'] = $value;
                        break;
                    case 'description':
                        $this->head['description'] = $value;
                        break;
                    default :
                        $this->head['meta'][$key] = $value;
                        break;
                }
            }
        }

        /**
         * 
         * @param type $name
         * @param type $path
         * @param type $isurl
         */
        public function addCss($name, $isurl = FALSE) {
            $path = $this->getCssFilePath($name);
            if ($path === FALSE) {
                $this->ci->logger->error("Template::addCss : Unknown file trying to add - $name");
            } else {
                if ($isurl === TRUE) {
                    $this->head['css'][$name] = $path;
                } else {
                    $css_path = 'css/' . $path . '.css';
                    $location = asset_path($css_path);
                    if (file_exists($location)) {
                        $this->head['css'][$name] = asset_url($css_path, TRUE);
                    } else {
                        $this->ci->logger->error("Template::addCss : File not found Location - $location");
                    }
                }
            }
        }

        /**
         * 
         * @param type $name
         * @param type $path
         * @param type $inHead
         * @param type $isurl
         */
        public function addJs($name, $inHead = FALSE, $isurl = FALSE) {
            $path = $this->getJsFilePath($name);
            if ($path === FALSE) {
                $this->ci->logger->error("Template::addJs : Unknown file trying to add - $name");
            } else {
                if ($isurl === TRUE) {
                    if ($inHead === TRUE) {
                        $this->head['js'][$name] = $path;
                    } else {
                        $this->js[$name] = $path;
                    }
                } else {
                    $js_path = 'js/' . $path . '.js';
                    $location = asset_path($js_path);
                    if (file_exists($location)) {
                        if ($inHead === TRUE) {
                            $this->head['js'][$name] = asset_url($js_path, TRUE);
                        } else {
                            $this->js[$name] = asset_url($js_path, TRUE);
                        }
                    } else {
                        $this->ci->logger->error("Template::addJs : File not found Location - $location");
                    }
                }
            }
        }

        /**
         * 
         * @param type $header
         */
        public function header($header = 'default') {
            switch ($header) {
                case 'blank':
                    $this->header = 'header_db_blank';
                    break;
                case 'login':
                    $this->header = 'header_db_login';
                    break;
                case 'admin':
                    $this->header = 'header_db_admin';
                    break;
                default :
                    $this->header = 'header_db_default';
                    break;
            }
        }

        /**
         * 
         * @param type $footer
         */
        public function footer($footer = 'default') {
            switch ($footer) {
                case 'blank':
                    $this->footer = 'footer_db_blank';
                    break;
                default :
                    $this->footer = 'footer_db_default';
                    break;
            }
        }

        /**
         * 
         * @param type $sidebar
         * @param type $template
         */
        public function sidebar($sidebar = '', $template = 'default') {
            switch ($sidebar) {
                case 'left':
                    $this->left_sidebar = $this->sidebarOption($template, $sidebar);
                    break;
                case 'right':
                    $this->right_sidebar = $this->sidebarOption($template, $sidebar);
                    break;
                case 'left-disable':
                    $this->left_sidebar = FALSE;
                    break;
                case 'right-disable':
                    $this->right_sidebar = FALSE;
                    break;
            }
        }

        /**
         * 
         */
        private function init() {
            $this->defaultCss();
            $this->defaultJs();
            $this->defaultMetaData();
            $this->header();
            $this->footer();
            $this->sidebar('left');
            $this->sidebar('right', FALSE);
        }

        /**
         * This will set the page name.
         * @param type $page_name
         */
        private function setPageName($page_name) {
            $this->ci->cisession->setCurrentPage($page_name);
        }

        /**
         * 
         * @param type $template
         * @param type $sidebar
         * @return string
         */
        private function sidebarOption($template = "", $sidebar = "") {
            $response = "";
            switch ($template) {
                case FALSE:
                    $response = FALSE;
                    break;
                default:
                    $response = 'sidebar_' . $sidebar . '_' . $template;
                    break;
            }
            return $response;
        }

        /**
         * 
         */
        private function defaultCss() {
            if (cssCompressionEnabled()) {
                $this->addCss('bootstrap-css');
                $this->addCss('fontawesome-css', TRUE);
                $compressedFiles = $this->ci->config->item('compress_css_files');
                if (is_array($compressedFiles) && count($compressedFiles) > 0) {
                    foreach ($compressedFiles as $filename => $path) {
                        $this->addCss($filename);
                    }
                }
            } else {
                $this->addCss('bootstrap-css');
                $this->addCss('fontawesome-css', TRUE);
                $this->addCss('style-css');
            }
        }

        private function defaultJs() {
            if (jsCompressionEnabled()) {
                $this->addJs('jquery-js');
                $this->addJs('popper-js');
                $this->addJs('bootstrap-js');
                $compressedFiles = $this->ci->config->item('compress_js_files');
                if (is_array($compressedFiles) && count($compressedFiles) > 0) {
                    foreach ($compressedFiles as $filename => $path) {
                        $this->addJs($filename);
                    }
                }
            } else {
                $this->addJs('jquery-js');
                $this->addJs('popper-js');
                $this->addJs('bootstrap-js');
                $this->addJs('app-js');
            }
        }

        private function getJsFilePath($name) {
            $files = array(
                'jquery-js' => 'vendor/jquery-3.4.1.min',
                'popper-js' => 'vendor/popper.min',
                'bootstrap-js' => 'vendor/bootstrap.min',
                'app-js' => 'js_app',
            );
            if (jsCompressionEnabled()) {
                $compressedFiles = $this->ci->config->item('compress_js_files');
                $files = (is_array($compressedFiles) && count($compressedFiles) > 0) ? array_merge($files, $compressedFiles) : $files;
            }
            return isset($files[$name]) ? $files[$name] : FALSE;
        }

        private function getCssFilePath($name) {
            $files = array(
                'bootstrap-css' => 'vendor/bootstrap.min',
                'fontawesome-css' => 'https://use.fontawesome.com/releases/v5.8.1/css/all.css',
                'style-css' => 'style',
            );
            if (cssCompressionEnabled()) {
                $compressedFiles = $this->ci->config->item('compress_css_files');
                $files = (is_array($compressedFiles) && count($compressedFiles) > 0) ? array_merge($files, $compressedFiles) : $files;
            }
            return isset($files[$name]) ? $files[$name] : FALSE;
        }

        /**
         * 
         */
        private function defaultMetaData() {
            $this->addMeta('title', 'Website Default Title');
            $this->addMeta('description', '');
        }

        /**
         * 
         * @param type $popupList
         * @return array
         */
        private function prepairPopupLocation($popupList = array()) {
            $popups = array();
            foreach ($popupList as $popupFileName) {
                $location = APPPATH . 'views/popup/' . $popupFileName . '.php';
                if (file_exists($location)) {
                    array_push($popups, $location);
                } else {
                    $this->ci->logger->error("Template::prepairPopupLocation : File not found Location - $location");
                }
            }
            return $popups;
        }

        /**
         * 
         * @param type $content
         */
        private function showPage($content) {
            switch (ENVIRONMENT) {
                case 'development':
                    echo $content;
                    break;
                default:
                    echo $this->sanitize($content);
                    break;
            }
        }

        /**
         * 
         * @param type $content
         */
        private function sanitize($content) {
            $search = array(
                '/\>[^\S ]+/s', // strip whitespaces after tags, except space
                '/[^\S ]+\</s', // strip whitespaces before tags, except space
                '/(\s)+/s', // shorten multiple whitespace sequences
                '/<!--(.|\s)*?-->/' // Remove HTML comments
            );

            $replace = array(
                '>',
                '<',
                '\\1',
                ''
            );
            echo preg_replace($search, $replace, $content);
        }

    }

}
```