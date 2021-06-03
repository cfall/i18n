# Weblate 

## Installation 
To avoid to install required dependencies you can install your own Weblate instance by using Docker. See [Weblate doc] (https://docs.weblate.org/en/latest/admin/install/docker.html) for more details. 

## Authentication 
Weblate does not support OpenID Connect authentication through docker configuration settings. You need to develop a custom python module to set it up. 
## Customization
There are two ways to customize weblate color scheme: change the color through Weblate interface and override the css files.<br>
1. Through Weblate interface <br>
In the menu click the administration button (spanner icon) then "Appearance". You will be shown different customization options like navigation color, navigation text color, hover color etc. Change the one you are interested in then click save. This is quite easy and simple but if you want to have more control over the color scheme customization you need to override css files. 

2. Override css files <br>
The static files coming with Weblate can be overriden. To change color scheme for example, you need to override the two css files (style-bootstrap.css and bootstrap.css) used by Weblate. The files should be placed into <strong>app/data/python/customize/static</strong> in the Docker volume.<br>
The placement of the Docker volume on host system depends on your Docker configuration, but usually it is stored in /var/lib/docker/volumes/weblate-docker_weblate-data/_data/. In the container it is mounted as /app/data.<br>
There are many ways to navigate to the docker volume if you are using Windows and Docker Desktop: <br>
<ul>
  <li>
    Open i18n_weblate CLI through Docker Desktop interface then type <code>cd /app/data/python/customize</code>
  </li>
  <li>
    Type <code>\\wsl$\docker-desktop-data\version-pack-data\community\docker\volumes\i18n_weblate-data\_data</code> in file explorer address bar
  </li>
</ul>
Below is where the css files should be placed for color scheme customization: 
<ul>
    <li>app/data/python/customize/static/style-bootstrap.css</b> for style-bootstrap.css</li>
    <li> app/data/python/customize/static/vendor/bootstrap/css/bootstrap.css</b> for bootstrap.css. Refer to [Weblate Docker repository] (https://github.com/WeblateOrg/weblate/tree/main/weblate/static) for default static files.</li>
</ul>
The files are copied to corresponding location on container startup, so restart is needed after changing the volume content.

## Translation 
To add translations, the steps below must be followed. <br>

1. Add project <br>
<p>In Weblate projects help to organize translations into logical set (for example to group all translations used within one application). To create a Project, you should click the plus (+) button in the menu then select "Add new translation project". Fill the form that is displayed then click save button. "Translation instructions" field is not mandatory so you can let it as is.</p>

2. Add component <br>
<p>In Weblate translations are organized into projects and components. Each project can contain number of components and those contain translations into individual languages. The component corresponds to one translatable file.<br>Once the project is created you will be taken to its dashboard. To create a component click "Add new translation component" button. You fill find many options shown in the component creation screen. Choose the one that suits best your case. For "Upload translations" option you will need to upload a zip file containing all your translation files. Fill in the form with other required info then click Continue button. On the next screen choose "File format JSON file, Filemask *.json". Click continue then save. When taken to the checklist screen just click "Return to the component" button. The component is created. <br>
If you choose "Start from scratch" option you will have to fill in the form and select the appropriate file format in the dropdown list then click Continue button. The component is created and you can click "Return to the component" button in the screen shown.</p> 

3. Add translations <br>
<p>To add translations click "Start new translation" button then choose a language in the list. On the language screen click "Add new translation string". Here you need to provide the translation key and the associated value. Next you have to click Add then save buttons. </p>

## Weblate as a Service 
<p>Weblate can be used to manage translations independently from application. This can permit another team for example to manage the translations. You have many options to do that. </p>

#### Use Weblate REST API
This option allows you to use REST API to retrieve the translations and use them in your application. In the following example, you will use Weblate as a Service for an Angular application. </p>
Prerequisites:
<ul>
    <li>Working angular app</li>
    <li>ngx translate installed in the app. See [GitHub repository] (https://github.com/ngx-translate/core) for more info.</li>
</ul>
<p>By default ngx translate uses the files in assets folder to translate the app. We will change this default behaviour by creating a custom loader that will fetch the translations from Weblate. The URL exposed to get the translations is <strong>weblate_domain_name/api/translations/(string:project)/(string:component)/(string:language)/file/</strong>. Since the test is done locally, weblate_domain_name is localhost. Let's say you want to load the english translations for your Service box project. By replacing the placeholder by actual values the URL becomes <strong>localhost/api/translations/service-box/gui/en/file/</strong>. gui is the name (Slug URL) of the component holding your project translations.</p>

1. Create the custom loader <br>
Create a typescript file containing your own translation loader.
```
export class TranslationHttpLoader implements TranslateLoader {
    constructor(private httpClient: HttpClient) {}
  
     public getTranslation(): Observable<any> {
      let observer = new Observable(observer => {
        this.httpClient.get('http://localhost:4200/api/translations/service-box/i18n/en/file/').subscribe(
          data => {
            observer.next(data);
            observer.complete();
          }
        );
      });
      return observer;
    }
  }
```

2. Configure the translation provider in App module <br>
In the App module you should define a HttpLoaderFactory:
```
export function HttpLoaderFactory(httpClient: HttpClient) {
    return new TranslationHttpLoader(httpClient);
}
```
and associate it with the provided TranslateLoader:
```
@NgModule({
  declarations: [
    AppComponent
  ],
  imports: [
    BrowserModule,
    HttpClientModule,
    TranslateModule.forRoot({
      loader: {
        provide: TranslateLoader,
        useFactory: HttpLoaderFactory,
        deps: [HttpClient]
      }
    })
  ],
  bootstrap: [AppComponent]
})
```

3. Use ngx translate pipe to get value <br>
To get the actual value for a key in the template use ngx translate as usual `{{'title' | translate}}` <br>
In this example the request to load the translations is sent to http://localhost:4200 because of the proxy set up to resolve CORS issues. 

#### Integrate Weblate with Version Control System (VCS)
To integrate Weblate with a VCS (GitHub for example) you choose "From control version" option in component creation screen. Fill in the form by specifying a Version Control System, the source code repository, the repository branch and other required information then click Continue. Before doing all this be sure the translation files are in the repository so that Weblate can find them after scanning the repository. Otherwise you will need to provide a base file that will be used. On the next screen select "File format JSON file, Filemask src/assets/i18n/*.json" then click Continue. You are taken to a new screen where you need to provide some information such as repository push URL, push branch. If you want to turn off push support you can let them empty. Let the other fields as is then click Save. On the next screen click "Return to the component". Weblate is now integrated with VCS. See [Weblate configuration] (https://docs.weblate.org/en/weblate-4.6.2/admin/projects.html#project-configuration) for more information about the VCS configuration. <br>
You can use git submodule for separating translations from source code while still having them under version control. See [Weblate FAQ] (https://docs.weblate.org/en/weblate-4.6.2/faq.html#faq-submodule) for more details.<br>
Weblate comes with native support for many VCS. To receive notifications on every push to the VCS repository, add the Weblate Webhook in the repository settings. See [Weblate documentation] (https://docs.weblate.org/en/weblate-4.6.2/admin/continuous.html#continuous-translation) for more details. 


## Notes
Weblate components are used to organize the translations. For example, you can have a component for the authentication part of your application and another one for the home page. But with this organization you will need to implement some logic to get all the app translations. So it's better to create one single component for all the translations. <br>
