---
Description: A dynamic-renderer plug-in is an object that displays the tablet pen data in real-time as it is being handled by the RealTimeStylus object.
ms.assetid: ac6d15f3-0917-4cc1-8c83-e34d3d063289
title: Dynamic-Renderer Plug-ins
ms.technology: desktop
ms.prod: windows
ms.author: windowssdkdev
ms.topic: article
ms.date: 05/31/2018
---

# Dynamic-Renderer Plug-ins

A dynamic-renderer plug-in is an object that displays the tablet pen data in real-time as it is being handled by the [**RealTimeStylus**](https://msdn.microsoft.com/en-us/library/ms704918(v=VS.85).aspx) object. Later, for events such as a form refresh, the dynamic-renderer plug-in or an ink-collector plug-in may redraw the ink.

## The DynamicRenderer Object

The [**RealTimeStylus**](https://msdn.microsoft.com/en-us/library/ms704918(v=VS.85).aspx) object implements the [**IStylusSyncPlugin**](https://msdn.microsoft.com/en-us/library/ms704312(v=VS.85).aspx) interface. The [**DynamicRenderer**](https://msdn.microsoft.com/en-us/library/ms701168(v=VS.85).aspx) object renders the ink in real-time, as it is being drawn. When the [**Refresh**](/windows/desktop/api/RTSCom/nf-rtscom-idynamicrenderer-refresh) method is called while the **DynamicRenderer** object is enabled, the **DynamicRenderer** object redraws the stroke currently being collected. The **DynamicRenderer** object's [**Enabled**](/windows/desktop/api/RTSCom/nf-rtscom-idynamicrenderer-get_enabled) property is initially set to **FALSE**.

> [!Note]  
> When calling the [**DynamicRenderer**](https://www.bing.com/search?q=**DynamicRenderer**) object's [**Refresh**](https://www.bing.com/search?q=**Refresh**) method from within a [Paint](https://www.bing.com/search?q=Paint) event handler in managed code, set the **DynamicRenderer** object's [**ClipRectangle**](https://www.bing.com/search?q=**ClipRectangle**) property to the [PaintEventArgs](https://www.bing.com/search?q=PaintEventArgs) object's [ClipRectangle](https://www.bing.com/search?q=ClipRectangle) property.

 

The [**DynamicRenderer**](https://msdn.microsoft.com/en-us/library/ms701168(v=VS.85).aspx) object can temporarily cache ink data. To use this feature in managed code, set the [EnableDataCache](https://www.bing.com/search?q=EnableDataCache) property to **TRUE**. When the [**DynamicRenderer**](https://www.bing.com/search?q=**DynamicRenderer**) object receives a call to its [**IStylusSyncPlugin.StylusUp**](https://www.bing.com/search?q=**IStylusSyncPlugin.StylusUp**) method, it caches the stroke data and adds custom stylus data to the Input queue in response to the [**StylusUpData**](https://www.bing.com/search?q=**StylusUpData**) object for the stroke. The [**CustomStylusData**](https://www.bing.com/search?q=**CustomStylusData**) object's [CustomDataId](https://www.bing.com/search?q=CustomDataId) property is set to the [DynamicRendererCachedDataGuid](https://www.bing.com/search?q=DynamicRendererCachedDataGuid) value, and the **CustomStylusData** object's Data property contains a DynamicRendererCachedData object. Call the **DynamicRenderer** object's [ReleaseCachedData](https://www.bing.com/search?q=ReleaseCachedData) method once the stroke has been collected and can be statically rendered. When the [**Refresh**](/windows/desktop/api/RTSCom/nf-rtscom-idynamicrenderer-refresh) method is called while the **DynamicRenderer** object is enabled, the **DynamicRenderer** object redraws all strokes that are cached. When the [**DataCacheEnabled**](/windows/desktop/api/RTSCom/nf-rtscom-idynamicrenderer-get_datacacheenabled) property is set to **false**, the cached stroke data is deleted.

The following diagram illustrates how the [**DynamicRenderer**](https://msdn.microsoft.com/en-us/library/ms701168(v=VS.85).aspx) object adds data to the tablet pen data when the **DynamicRenderer** object's [**DataCacheEnabled**](/windows/desktop/api/RTSCom/nf-rtscom-idynamicrenderer-get_datacacheenabled) property is set.

![illustration showing the dynamicrenderer data flow](images/75f4ee7b-160c-410e-bfae-dfc676a9829c.gif)

In this diagram the circle lettered "SD" represents a [**StylusDown**](/windows/desktop/api/RTSCom/nf-rtscom-istylusplugin-stylusdown) object and the circles lettered "P" represent [**Packets**](/windows/desktop/api/RTSCom/nf-rtscom-istylusplugin-packets) objects that have already been added to the [**RealTimeStylus**](https://msdn.microsoft.com/en-us/library/ms704918(v=VS.85).aspx) object's output queue and that have not yet been sent to the asynchronous plug-in collection. The circle lettered "SU" represents a [**StylusUp**](/windows/desktop/api/RTSCom/nf-rtscom-istylusplugin-stylusup) object that the **RealTimeStylus** object is currently processing. It is sent to the synchronous plug-in collection and then placed on the output queue. The circles lettered "DR" represent custom stylus data that is added to the input queue by the [**DynamicRenderer**](https://msdn.microsoft.com/en-us/library/ms701168(v=VS.85).aspx) plug-in in response to the stylus up notification associated with "SU". The custom stylus data lettered "DR" is then passed to the synchronous plug-ins and then to the output queue before the next tablet pen data is processed. The empty circle represents the position in the output queue where future tablet pen data is added. Also represented on the diagram is the ink-collector plug-in calling the **DynamicRenderer** object's [**ReleaseCachedData**](/windows/desktop/api/RTSCom/nf-rtscom-idynamicrenderer-releasecacheddata) method to release the cached stroke data after the ink-collection plug-in has processed the stroke.

### Special Considerations

The following list describes other points to take into consideration when using the [**DynamicRenderer**](https://msdn.microsoft.com/en-us/library/ms701168(v=VS.85).aspx) object.

-   You should not attach a [**DynamicRenderer**](https://msdn.microsoft.com/en-us/library/ms701168(v=VS.85).aspx) object to more than one [**RealTimeStylus**](https://msdn.microsoft.com/en-us/library/ms704918(v=VS.85).aspx) object. Once two **RealTimeStylus** objects to which the **DynamicRenderer** object is attached are enabled, the following occurs.

    -   The [**DynamicRenderer**](https://msdn.microsoft.com/en-us/library/ms701168(v=VS.85).aspx) object throws an exception in response to the second call to its [**RealTimeStylusEnabled**](/windows/desktop/api/RTSCom/nf-rtscom-istylusplugin-realtimestylusenabled) method.
    -   The second [**RealTimeStylus**](https://msdn.microsoft.com/en-us/library/ms704918(v=VS.85).aspx) object that was enabled generates an [**Error**](/windows/desktop/api/RTSCom/nf-rtscom-istylusplugin-error) object and notifies the remaining plug-ins in its plug-in collections of the error.
    -   The [**DynamicRenderer**](https://msdn.microsoft.com/en-us/library/ms701168(v=VS.85).aspx) object stops rendering tablet pen data.

-   The [**RealTimeStylus**](https://msdn.microsoft.com/en-us/library/ms704918(v=VS.85).aspx) object throws an exception when its [**AddCustomStylusDataToQueue**](/windows/desktop/api/RTSCom/nf-rtscom-irealtimestylus-addcustomstylusdatatoqueue) method is called with the *guid* parameter set to the DynamicRendererCachedDataGuid globally unique identifier (GUID).
-   The [**DynamicRenderer**](https://msdn.microsoft.com/en-us/library/ms701168(v=VS.85).aspx) object is implemented as a Component Object Model (COM) wrapper, and you cannot call its [**IStylusSyncPlugin**](https://msdn.microsoft.com/en-us/library/ms704312(v=VS.85).aspx) interface methods directly. For more information about COM operation and the [**RealTimeStylus**](https://msdn.microsoft.com/en-us/library/ms704918(v=VS.85).aspx) object, see [Implementation Notes for the StylusInput APIs](implementation-notes-for-the-stylusinput-apis.md).

## Custom Rendering

You can create your own dynamic-renderer plug-in by creating a synchronous plug-in that subscribes to the [**StylusDown**](/windows/desktop/api/RTSCom/nf-rtscom-istylusplugin-stylusdown), [**Packets**](/windows/desktop/api/RTSCom/nf-rtscom-istylusplugin-packets), and [**StylusUp**](/windows/desktop/api/RTSCom/nf-rtscom-istylusplugin-stylusup) notifications. Your plug-in can then render the stroke as it is being drawn. This may be one way of implementing a selection tool that uses a free-form selection or selection box, for example.

 

 


