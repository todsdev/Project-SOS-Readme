<!-- # Title -->
# Panic Button
  ![Demo](https://media.discordapp.net/attachments/1101143996400676924/1101144154727251988/logo2.png?width=1000&height=1000)


---

 <!-- # Short Description -->

> The app was designed to create a mechanism that allows the user to send data to their contacts in possible dangerous moments.

> It works under two different pespectives: 
>- The **Contacts**, those who receive a SMS when the emergency button is pressed, with a standard/personalised message and the last location included. 
>- The **Protectors** are a different category, which who you add in this list, has access to the user's last known locations.

It was developed to allow users to have a safe interaction with their own contacts in moments of possible emergency. It allows the user to access the app functionalities in several ways:
>- **Broadcast** - It records the volume up/down from the system in order to access the functionality to send messages. It allow the user to discreetly trigger the SMS system. 
>- **Widget** - The app contains a widget that allows the user to click in it and also trigger the SMS system. 
>- **In-app Button** - The user can only clicks in the button on the app and trigger the SMS system as well. 

<small><em>The application was fully designed in Brazilian Portuguese.</em></small>


<!-- # Badges -->
<div style="display: inline_block"><br>
    <img height="30" width="40" src="https://cdn.jsdelivr.net/gh/devicons/devicon/icons/androidstudio/androidstudio-original.svg">
    <img height="30" width="40" src="https://cdn.jsdelivr.net/gh/devicons/devicon/icons/kotlin/kotlin-original.svg">
    <img height="30" width="40" src="https://cdn.jsdelivr.net/gh/devicons/devicon/icons/firebase/firebase-plain.svg">
    <img height="30" width="40" src="https://cdn.jsdelivr.net/gh/devicons/devicon/icons/git/git-original.svg">
    <img height="30" width="40" src="https://cdn.jsdelivr.net/gh/devicons/devicon/icons/github/github-original.svg">

</div>

---

# Tags

`Android Studio` `Kotlin` `Coroutines` `MVVM` `Room` `Hilt` `Clean Architecture` `Firebase Authentication` `Firebase Realtime Database` `Remote Config` `Worker` `Broadcast` `Deep Link`

---


# Demo

![](https://cdn.discordapp.com/attachments/1101143996400676924/1101206848947892285/introduction.gif)

Starts the app with a full introduction of the app functionalities.

***
<img src="https://cdn.discordapp.com/attachments/1101143996400676924/1101208701194481734/authentication.gif" width="300" height="649"/>

All the authentication features:
>- *Login* and *Register* with **email and password** and **authentication providers**
>- *Recover* lost password

***
![](https://cdn.discordapp.com/attachments/1101143996400676924/1101206849925165076/permissions.gif)

Requesting permission and handling scenarios where it is denied.

***
![](https://cdn.discordapp.com/attachments/1101143996400676924/1101215875916382308/in_app_view_and_funcionalities.gif)

Small app functionalities:
>- Control the view of your current location
>- Check your settings to edit your **message** and **app** configurations

***
| Contacts | Protectors |
| --- | --- |
| <img src="https://cdn.discordapp.com/attachments/1101143996400676924/1101215876360970380/importing_and_adding_contacts.gif" width="300" height="649"/> | <img src="https://cdn.discordapp.com/attachments/1101143996400676924/1101215875459194931/switching_contact_and_protector_editing_protector.gif" width="300" height="649"/> |
| Receives the SMS with user's last known location  | Receives constantly the user's last known location |

**Contacts** and **Protectors** allows two different functionalities but they are easily added, created or deleted from the tabs next to each other. 

***
![](https://cdn.discordapp.com/attachments/1101143996400676924/1101223961141985320/triggering_button.gif)

Clicking in the button to start the countdown that, if not stopped, will trigger the SMS sending action. It allows to use the app and, in case of miss click, it's easily fixable and no warning is sended to your contacts. The time used in the countdown can be edited in the *Settings* page. 

***
| Adding Protector | Receiving Protected |
| --- | --- |
| <img src="https://cdn.discordapp.com/attachments/1101143996400676924/1101255022714556587/1.gif" width="300" height="649"/> | <img src="https://cdn.discordapp.com/attachments/1101143996400676924/1101255023100444752/2.gif" width="300" height="649"/> |
| Adding a **Protector** | Receiving the **Protected** information |

---

# Code Example
```kotlin
@HiltWorker
class ApplicationWorker @AssistedInject constructor(
    private val useCases: ApplicationUseCases,
    @Assisted context: Context,
    @Assisted parameters: WorkerParameters): CoroutineWorker(context, parameters) {

    override suspend fun doWork(): Result {
        return try {
            var workResult: Result? = null
            withContext(Dispatchers.IO) {
                useCases.requestLocationUpdate().collectLatest { result ->
                    when(result) {
                        is ResourceState.Success -> {
                            result.data?.let { location ->
                                val address = applicationContext.getAddressFromLocation(location.latitude, location.longitude)
                                useCases.insertLastLocation(location)
                                useCases.insertLocationRoom(
                                    FallbackLocationModel(
                                        latitude = location.latitude,
                                        longitude = location.longitude,
                                        timestamp = System.currentTimeMillis(),
                                        address = address
                                    )
                                )
                            }
                            configCheckRoomLastLocations()
                            workResult = Result.success()
                        }
                        else -> { workResult = Result.retry() }
                    }
                }
            }
            workResult ?: Result.retry()
        } catch(e: Exception) {
            Result.retry()
        }
    }
}
```

This snippet above shows the **Coroutine Worker** class, which allows the application to runs in the Android system background. It is used with **Hilt Work**, that allows the app to inject dependencies of my *ApplicationUseCases* class and access repository functions. The worker class allows to retrieve the user last location every few time (about 15 minutes). The location have two different destinies:
>- [Google Firebase](https://firebase.google.com) - Allow the **Protectors** to access the user's last known location.
>- [ROOM](https://developer.android.com/jetpack/androidx/releases/room?hl=pt-br) - Creates a fallback for, in case the last location is needed and the system is not able to provide it right in the moment, it will retrieve the last known location from the local database in order to access the app functions.

---

# Libraries

>- [Timber](https://github.com/JakeWharton/timber)
>- [Lifecycle](https://developer.android.com/jetpack/androidx/releases/lifecycle)
>- [Coroutines](https://developer.android.com/kotlin/coroutines?hl=pt-br)
>- [KTX](https://developer.android.com/kotlin/ktx)
>- [Navigation Components](https://developer.android.com/guide/navigation)
>- [Hilt](https://dagger.dev/hilt/)
>- [Hilt Work](https://developer.android.com/training/dependency-injection/hilt-jetpack?hl=pt-br)
>- [ROOM](https://developer.android.com/jetpack/androidx/releases/room?hl=pt-br)
>- [Worker](https://developer.android.com/reference/androidx/work/Worker)
>- [Material Intro](https://github.com/heinrichreimer/material-intro)
>- [Google Firebase](https://firebase.google.com) - *Realtime Database*, *Authentication*, *Remote Config*
---

# Contributors

- [Thiago Rodrigues](https://www.linkedin.com/in/tods/) - Coding
- [Raphael Marones](https://github.com/RaphaelMarones) - Review
