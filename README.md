# bookstack-pdf-viewer
Step by step to setup pdf-viewer for bookstack app

## Step #1
Download PDFJS prebuilt version from https://mozilla.github.io/pdf.js/getting_started/#download
Unzip the file to 'pdfjs' folder and move it to the BookStack app 'public' folder. Ensure both 'build' and 'web' folders and its files are there.

## Step #2 configure bookstack
All the below steps assume that you are using docker for the bookstack deployment 

### 2.1 copy the pdfjs folder to public folder 
```bash
docker cp pdfjs bookstack:/app/www/public
docker exec -it bookstack bash
#This is a workaround step to fix an issue with web/viewer.html, it intends to include pdf.worker.js not pdf.worker.mjs
cd /app/www/public/pdfjs/build/
cp pdf.worker.mjs pdf.worker.js
```
![image](https://github.com/ddkhanh/bookstack-pdf-viewer/assets/5151868/e83e1069-e81f-48ec-8a1e-066fa21f5a8e)

### 2.2 Config nginx to treat the mjs file as javascript 
All of the below steps should be executed in bookstack docker container

```bash
vi /etc/nginx/mime.types
```
Replace this line

From
```
application/javascript                           js;
```
To
```
application/javascript                           js mjs;
```
Then reload the nginx config
```bash
/usr/sbin/nginx -s reload
```

### 2.3 Copy and paste the following code to Custom HTML Head Content field on Settings page -> Customization

```html
<script type="text/javascript">

  // ------------------- THIS SECTION ADDS A PDF BUTTON TO THE EDITOR TOOLBAR THAT ALLOWS YOU TO EMBED PDFS 

  // Use BookStack editor event to add custom "Insert PDF" button into main toolbar
  window.addEventListener('editor-tinymce::pre-init', event => {
      const mceConfig = event.detail.config;
      mceConfig.toolbar = mceConfig.toolbar.replace('link', 'link insertpdf')
  });

  // Use BookStack editor event to define the custom "Insert PDF" button.
  window.addEventListener('editor-tinymce::setup', event => {
    const editor = event.detail.editor;

    // Add PDF insert button
    editor.ui.registry.addButton('insertpdf', {
      tooltip: 'Insert PDF',
      icon: 'document-properties',
      onAction() {
        editor.windowManager.open({
          title: 'Insert PDF',
          body: {
            type: 'panel',
            items: [
              {type: 'textarea', name: 'pdfurl', label: 'PDF URL'}
            ]
          },
          onSubmit: function(e) {
            // Insert content when the window form is submitted
            editor.insertContent(`<iframe id="pdf-viewer" title="PDF viwer" src="/pdfjs/web/viewer.html?file=${e.getData().pdfurl}" width="100%" height="450vh"></iframe>`);
            e.close();
          },
          buttons: [
            {
              type: 'submit',
              text: 'Insert PDF'
            }
          ]
        });
      }
    });

  });
</script>
```
#### Insert PDF Button


![image](https://github.com/ddkhanh/bookstack-pdf-viewer/assets/5151868/ada4c966-8332-42aa-bc88-78560051e31a)

#### Insert PDF Form input

![image](https://github.com/ddkhanh/bookstack-pdf-viewer/assets/5151868/7b704a7f-1e99-4b2d-8517-cd55798f50dc)

#### Insert PDF Editor

![image](https://github.com/ddkhanh/bookstack-pdf-viewer/assets/5151868/7a51a2c9-6584-4b97-8606-cf6fd003d4aa)

#### PDF Viewer

![image](https://github.com/ddkhanh/bookstack-pdf-viewer/assets/5151868/3b7efd2e-f413-48fb-a148-ab05e821c761)

# Appendix 
## Persit changes
If you want to preserve the changes, which is helpful for those who want to regularly upgrade the BookStack image, please follow the instructions mentioned here

[How to deal with problem on docker image update](https://github.com/ddkhanh/bookstack-pdf-viewer/issues/1#issuecomment-2048845117)

## Shadow effect
A small tweak if you want to strengthen the shadow to make the page appear closer to the reader and focus more on the content

```css
<style>
.page-content clearfix {
 font-size: 14px;
}
#bkmrk-page-title {
    font-size: 35px;
}
.card {
    background-color: #fff;
    box-shadow: 0 20px 60px -1px rgba(0,0,0,.1);
    border-radius: 10px;
    border: 1px solid transparent;
}


.tri-layout-middle-contents {
    max-width: unset;
}
</style>
```

![image](https://github.com/ddkhanh/bookstack-pdf-viewer/assets/5151868/7f865134-2ef6-44ff-90f0-6c29f62804f5)


