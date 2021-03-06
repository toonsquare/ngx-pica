# ngx-tooning-pica

>ngx-tooning-pica is an Angular (LTS) module to resize images files in ionic webview  using <a href="https://github.com/nodeca/pica">pica - high quality image resize in browser</a>.

> ionic has <a href="https://github.com/ionic-team/ionic-native/issues/505">FileReader issue</a>

[![latest](https://img.shields.io/npm/v/ngx-tooning-pica)](https://www.npmjs.com/package/ngx-tooning-pica) 

## Important
ngx-tooning-pica Angular 5 compatibility is under version **1.1.8**  
```bash
$ npm install ngx-tooning-pica --save
```

## Install
1. Add `ngx-tooning-pica` module as dependency to your project.
```bash
$ npm install pica exifr ngx-tooning-pica --save
```
2. Include `NgxPicaModule` into your main AppModule or in module where you will use it.
```
// app.module.ts
import { NgModule } from '@angular/core';
import { BrowserModule } from '@angular/platform-browser';
import { NgxPicaModule } from 'ngx-tooning-pica';

@NgModule({
  imports: [
    BrowserModule,
    NgxPicaModule
  ],
  declarations: [ AppComponent ],
  exports: [ AppComponent ]
})
export class AppModule {}
```



## Services
* **NgxPicaService** - Manipulate images using <a href="https://github.com/nodeca/pica">pica - high quality image resize in browser</a>
* **NgxPicaImageService** - Supplementary services to help you work with images

## NgxPicaService Methods
#### `.resizeImages(files: File[], width: number, height: number, options?: NgxPicaResizeOptionsInterface): Observable<File>`
This method resize an array of images doing it sequentially to optimize CPU and memory use.
* **files:[]** - Array of images to resize
* **width** - Width to be resized (px)
* **height** - Height to be resized (px)
* **options** - Based on <a href="https://github.com/nodeca/pica#resizefrom-to-options---promise">pica - resize options</a>, we've also added exif and aspect ratio options:
    * **exifOptions**:    
        * **forceExifOrientation** - [default: true] Set false to avoid image orientation from Exif info.
    * **aspectRatio**:     
        * **keepAspectRatio** - Set true to ensure the aspect ratio of the image is maintained as it get resized
        * **forceMinDimensions** - Set true to ensure the dimensions of image resized will be greater than width and height values defined

The Observable receives a next on every file that has been resized.
If something goes wrong the Observable receive an error.

All errors are wrapped by NgxPicaErrorInterface.

### `.resizeImage(file: File, width: number, height: number, options?: NgxPicaResizeOptionsInterface): Observable<File>`
Same as above but only takes one file instead of an array of files.

### `.compressImages(files: File[], sizeInMB: number, options?: NgxPicaCompressOptionsInterface): Observable<File>`
This method compress an array of images doing it sequentially to optimize CPU and memory use.
* **files:[]** - Array of images to resize
* **sizeInMB** - File size in MegaBytes
* **options** - Same as resize options, but only for exif orientation:
    * **exifOptions**:    
        * **forceExifOrientation** - [default: true] Set false to avoid image orientation from Exif info.

The Observable receives a next on every file that has been resized.
If something goes wrong the Observable receive an error.

All errors are wrapped by NgxPicaErrorInterface.

### `.compressImage(file: File, sizeInMB: number, options?: NgxPicaCompressOptionsInterface): Observable<File>`
Same as above but only takes one file instead of an array of files.

## NgxPicaImageService Methods
#### `.isImage(file: File): boolean`
This method check if a file is an image or not

## Data Structures
```
export interface ExifOptions {
  forceExifOrientation: boolean;
}
```

```
export interface AspectRatioOptions {
    keepAspectRatio: boolean;
    forceDimensions?: boolean;
}
```

```
export interface NgxPicaResizeOptionsInterface {
    exifOptions: ExifOptions;
    aspectRatio?: AspectRatioOptions;
    quality?: number;
    alpha?: boolean;
    unsharpAmount?: number;
    unsharpRadius?: number;
    unsharpThreshold?: number;
}
```

```
export interface NgxPicaCompressOptionsInterface {
  exifOptions: ExifOptions;
}
```

```
export enum NgxPicaErrorType {
    NO_FILES_RECEIVED = 'NO_FILES_RECEIVED',
    CANVAS_CONTEXT_IDENTIFIER_NOT_SUPPORTED = 'CANVAS_CONTEXT_IDENTIFIER_NOT_SUPPORTED',
    NOT_BE_ABLE_TO_COMPRESS_ENOUGH = 'NOT_BE_ABLE_TO_COMPRESS_ENOUGH'
}

export interface NgxPicaErrorInterface {
    err: NgxPicaErrorType;
    file?: File;
}
```

## Example


```ts
import { Component, EventEmitter } from '@angular/core';
import { NgxPicaService } from 'ngx-tooning-pica';

@Component({
  selector: 'app-home',
  template: `
      <img *ngFor="let image of images" [src]="image" />
  
      <input type="file" [attr.accept]="image/*" multiple
             (change)="handleFiles($event)">
  `
})
export class AppHomeComponent {
    images: File[] = [];
    
    constructor(private _ngxPicaService: NgxPicaService) {
    
    }
    
    public handleFiles(event: any) {
        const files: File[] = event.target.files;
        
        this._ngxPicaService.resizeImages(files, 1200, 880)
            .subscribe((imageResized: File) => {
                let reader: FileReader = new FileReader();
                
                reader.addEventListener('load', (event: any) => {
                    this.images.push(event.target.result);
                }, false);
                
                reader.readAsDataURL(imageResized);
                
            }, (err: NgxPicaErrorInterface) => {
                throw err.err;
            });
    }
```  
