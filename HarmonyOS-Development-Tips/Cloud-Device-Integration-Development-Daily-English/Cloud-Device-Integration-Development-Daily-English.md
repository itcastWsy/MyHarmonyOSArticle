# Cloud-Device Integration Development: Daily English Quote

## Background

In today's increasingly accelerated globalization, English has become an important bridge connecting the world. The "Daily English Quote" application is your pocket English learning expert, dedicated to helping you improve your English abilities in the most convenient, interesting, and efficient way.

## Effect

![image-20241210075206578](%E7%AB%AF%E4%BA%91%E4%B8%80%E4%BD%93%E5%8C%96%E5%BC%80%E5%8F%91%20%E6%AF%8F%E6%97%A5%E4%B8%80%E5%8F%A5%E8%8B%B1%E8%AF%AD.assets/image-20241210075206578.png)

## Technical Highlights

1. Cloud Functions
2. Cloud Database
3. Cloud Storage
4. AppLink
5. AtomicServiceNavigation - Atomic Service Navigation
6. AtomicServiceTabs - Atomic Service Tabs
7. FunctionalButton - Get User Avatar
8. Silent Login - No user awareness required to get user unionID

## Business Flow

![image-20241210075359617](%E7%AB%AF%E4%BA%91%E4%B8%80%E4%BD%93%E5%8C%96%E5%BC%80%E5%8F%91%20%E6%AF%8F%E6%97%A5%E4%B8%80%E5%8F%A5%E8%8B%B1%E8%AF%AD.assets/image-20241210075359617.png)

## Project Structure

![image-20241210075612734](%E7%AB%AF%E4%BA%91%E4%B8%80%E4%BD%93%E5%8C%96%E5%BC%80%E5%8F%91%20%E6%AF%8F%E6%97%A5%E4%B8%80%E5%8F%A5%E8%8B%B1%E8%AF%AD.assets/image-20241210075612734.png)

```
├─components
│      CloudImage.ets
│      EgCollectIcon.ets
│      EgLikeIcon.ets
│
├─entryability
│      EntryAbility.ets
│
├─pages
│  │  CloudFunction.ets
│  │  CloudStorage.ets
│  │  Index.ets
│  │
│  └─CloudDb
│          BookInfo.ets
│          CloudDb.ets
│          DbInsert.ets
│          Post.ts
│
├─services
│      userServices.ets
│      wallPaperServices.ets
│
├─tabPages
│      AboutTab.ets
│      HomeTab.ets
│      ListTab.ets
│
├─types
│  │  index.ets
│  │
│  └─Db
│          userDbEntry.ets
│          wallpaperDbEntry.ets
│
├─utils
│      auth.ets
│      cloudDbHelper.ets
│      cloudFuncHelper.ets
│      cloudStorageHelper.ets
│      constant.ets
│      hobby.ets
│      index.ets
│      time.ets
│
└─views
        DetailView.ets
        LikeCollectVIew.ets
        UserView.ets
```

## Database Design Overview

### User

User Management

![image-20241210075854044](%E7%AB%AF%E4%BA%91%E4%B8%80%E4%BD%93%E5%8C%96%E5%BC%80%E5%8F%91%20%E6%AF%8F%E6%97%A5%E4%B8%80%E5%8F%A5%E8%8B%B1%E8%AF%AD.assets/image-20241210075854044.png)

### Wallpaper

Wallpaper Management

![image-20241210075920026](%E7%AB%AF%E4%BA%91%E4%B8%80%E4%BD%93%E5%8C%96%E5%BC%80%E5%8F%91%20%E6%AF%8F%E6%97%A5%E4%B8%80%E5%8F%A5%E8%8B%B1%E8%AF%AD.assets/image-20241210075920026.png)

## Cloud Storage

![image-20241210080008459](%E7%AB%AF%E4%BA%91%E4%B8%80%E4%BD%93%E5%8C%96%E5%BC%80%E5%8F%91%20%E6%AF%8F%E6%97%A5%E4%B8%80%E5%8F%A5%E8%8B%B1%E8%AF%AD.assets/image-20241210080008459.png)

## Cloud Functions

The cloud functions here use **Cloud Objects**, managing various business functions of an entity using object methods.

![image-20241210080040849](%E7%AB%AF%E4%BA%91%E4%B8%80%E4%BD%93%E5%8C%96%E5%BC%80%E5%8F%91%20%E6%AF%8F%E6%97%A5%E4%B8%80%E5%8F%A5%E8%8B%B1%E8%AF%AD.assets/image-20241210080040849.png)

Cloud Objects Overview

![image-20241210080646917](%E7%AB%AF%E4%BA%91%E4%B8%80%E4%BD%93%E5%8C%96%E5%BC%80%E5%8F%91%20%E6%AF%8F%E6%97%A5%E4%B8%80%E5%8F%A5%E8%8B%B1%E8%AF%AD.assets/image-20241210080646917.png)

## AppLink

Quickly open other atomic services

![image-20241210080847820](%E7%AB%AF%E4%BA%91%E4%B8%80%E4%BD%93%E5%8C%96%E5%BC%80%E5%8F%91%20%E6%AF%8F%E6%97%A5%E4%B8%80%E5%8F%A5%E8%8B%B1%E8%AF%AD.assets/image-20241210080847820.png)

## AtomicServiceNavigation - Atomic Service Navigation

![image-20241210080912288](%E7%AB%AF%E4%BA%91%E4%B8%80%E4%BD%93%E5%8C%96%E5%BC%80%E5%8F%91%20%E6%AF%8F%E6%97%A5%E4%B8%80%E5%8F%A5%E8%8B%B1%E8%AF%AD.assets/image-20241210080912288.png)

## AtomicServiceTabs - Atomic Service Tabs

![image-20241210080939232](%E7%AB%AF%E4%BA%91%E4%B8%80%E4%BD%93%E5%8C%96%E5%BC%80%E5%8F%91%20%E6%AF%8F%E6%97%A5%E4%B8%80%E5%8F%A5%E8%8B%B1%E8%AF%AD.assets/image-20241210080939232.png)

![image-20241210081006366](%E7%AB%AF%E4%BA%91%E4%B8%80%E4%BD%93%E5%8C%96%E5%BC%80%E5%8F%91%20%E6%AF%8F%E6%97%A5%E4%B8%80%E5%8F%A5%E8%8B%B1%E8%AF%AD.assets/image-20241210081006366.png)

## FunctionalButton

![image-20241210081024984](%E7%AB%AF%E4%BA%91%E4%B8%80%E4%BD%93%E5%8C%96%E5%BC%80%E5%8F%91%20%E6%AF%8F%E6%97%A5%E4%B8%80%E5%8F%A5%E8%8B%B1%E8%AF%AD.assets/image-20241210081024984.png)

![image-20241210081048299](%E7%AB%AF%E4%BA%91%E4%B8%80%E4%BD%93%E5%8C%96%E5%BC%80%E5%8F%91%20%E6%AF%8F%E6%97%A5%E4%B8%80%E5%8F%A5%E8%8B%B1%E8%AF%AD.assets/image-20241210081048299.png)

## Silent Login

When entering the homepage, directly perform silent login to implement user information recording with minimal code.

![image-20241210081136874](%E7%AB%AF%E4%BA%91%E4%B8%80%E4%BD%93%E5%8C%96%E5%BC%80%E5%8F%91%20%E6%AF%8F%E6%97%A5%E4%B8%80%E5%8F%A5%E8%8B%B1%E8%AF%AD.assets/image-20241210081136874.png)

![image-20241210081153362](%E7%AB%AF%E4%BA%91%E4%B8%80%E4%BD%93%E5%8C%96%E5%BC%80%E5%8F%91%20%E6%AF%8F%E6%97%A5%E4%B8%80%E5%8F%A5%E8%8B%B1%E8%AF%AD.assets/image-20241210081153362.png)

## Calling Cloud Functions

Call cloud functions to store user unionID for user information recording.

![image-20241210081227610](%E7%AB%AF%E4%BA%91%E4%B8%80%E4%BD%93%E5%8C%96%E5%BC%80%E5%8F%91%20%E6%AF%8F%E6%97%A5%E4%B8%80%E5%8F%A5%E8%8B%B1%E8%AF%AD.assets/image-20241210081227610.png)

# Summary

The "Daily English Quote" application carefully selects practical English golden sentences daily, covering diverse fields to help accumulate vocabulary and expressions. It provides in-depth analysis of sentence grammar, vocabulary expansion, and background with examples to help users thoroughly understand and internalize. It also offers diverse learning modes, adapting to different habits and fragmented time, starting your English learning improvement journey in a convenient, interesting, and efficient manner.
