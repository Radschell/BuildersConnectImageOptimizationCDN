# Builders Connect 2026 — Exercise 1 Guide: Cloud CDN & Image Optimization (A to Z)

&nbsp;

> **Goal:** Go from an uncached Cloud Storage (GCS) bucket to globally cached content delivery across **100+ Google Cloud points of presence (PoPs)** with automated **Edge Image Optimization** in minutes.

## **Important Links**

1. **Claim Demo Account with Credits:** Paste the URL [https\://g.dev/credits/builder-connect-warsaw-cdn](https://g.dev/credits/builder-connect-warsaw-cdn)  
2. **Supported Image Optimization Operations (`?imop=`) Docs:** [https\://docs.cloud.google.com/cdn/docs/image-opt-supportedops](https://docs.cloud.google.com/cdn/docs/image-opt-supportedops)  
3. **Cloud CDN Documentation:** [https\://cloud.google.com/cdn/docs/overview](https://cloud.google.com/cdn/docs/overview)  
4. **Global External Application Load Balancer (GCLB) Docs:** [https\://cloud.google.com/load-balancing/docs/https](https://cloud.google.com/load-balancing/docs/https)  
5. **GKE Gateway Demo Repository:** [https\://github.com/burczyk/gke-cdn-demo](https://github.com/burczyk/gke-cdn-demo)  
6. **GKE Gateway Documentation:** [https\://cloud.google.com/kubernetes-engine/docs](https://cloud.google.com/kubernetes-engine/docs)  
7. **Google Cloud Console:** [https\://console.cloud.google.com](https://console.cloud.google.com)

## **Part 1: Step-by-Step Cloud CDN & Image Optimization Setup (Starting from Cloud CDN UI)**

In Exercise 1, we start directly in the **Cloud CDN** section of the Google Cloud Console to create a **Cloud CDN origin**, a **private Cloud Storage (GCS) bucket** (with Private Bucket Access), and a **Global External Application Load Balancer** in one unified workflow—then upload **`premium-asset.jpg`** and enable **Image Optimization** on the default route.

&nbsp;


### **Step 1: Start in Cloud CDN & Create a Bucket \+ Load Balancer**

1. In the Google Cloud Console, navigate to **Network Services → Cloud CDN** and click **\+ Add origin**.  
2. **Origin basics (Create a New Bucket):**  
   * **Origin type:** Select **Backend bucket**.  
   * **Define your backend bucket:** Select **New backend bucket**.  
   * **Cloud Storage bucket:** Click **Browse**, then click the **Create new bucket** (`+`) icon at the top of the panel:  
     * Enter a globally unique bucket name (for example, `cdn-demo-bucket`) and click **Continue**.  
     * Under **Choose how to control access to objects**, keep **Enforce public access prevention on this bucket** checked (so the bucket remains private) and click **Create**, then click **Select**.  
   * **Origin name:** Enter a name for your CDN origin (for example, `media-bucket-origin`) and click **Next**.  
3. **Host and path rules (Create a New Load Balancer):**  
   * Under **Attach a load balancer**, select **Create new load balancer for me** *(Google Cloud automatically configures an IPv4/IPv6 Global External Application Load Balancer and attaches your bucket to the default route)*.  
   * Enter a **Load Balancer name** (for example, `cdn-demo-lb`) and click **Next**.  
4. **Cache performance:**  
   * Set **Cache mode** to **Force cache all content** (recommended for Private Bucket Access so private bucket responses are cached) and **Default TTL** (1 hour), and click **Done**.

### **Step 2: Configure Private Bucket Access & Upload the Image**

1. Navigate to **Cloud Storage → Buckets** and click your newly created bucket (`cdn-demo-bucket`).  
2. **Grant Private Bucket Access to the Cloud CDN service agent:**  
   * Open the **Permissions** tab and click **Grant access**.  
   * Set **New principals** to `service-<PROJECT_NUMBER>@https-lb.iam.gserviceaccount.com`, select the role **Storage → Storage Object Viewer** (replace \<PROJECT\_NUMBER\> with your project number from the Cloud Console Welcome page), and click **Save**.  
3. **Upload the master image:**  
   * Open the **Objects** tab, click **Upload files**, and upload premium-asset.jpg to the root of your bucket.

### **Step 3: Enable Image Optimization & Test Delivery**

1. Navigate to **Network Services → Load balancing** (or open the **URL maps** tab in **Network Services → Cloud CDN**), click your load balancer (`cdn-demo-lb`), and click **Edit**.  
2. Click **Routing rules** in the left-hand menu:  
   * On the **default route** (`All unmatched (default)`), expand **Image optimization** and check **Enable image optimization** (*"Optimize images to improve website performance and reduce costs"*).  
   * Click **Update** to apply the change.  
3. Copy the load balancer's IPv4 or IPv6 address (`<LB_IP>`) from the **Load balancer details** page (allow **2–3 minutes** for initial propagation across Google's 100+ edge PoPs).  
4. Test baseline image delivery, Cloud CDN caching, and your first **Image Optimization** transformation in your browser (with **Chrome DevTools → Network** open): first visit http\://\<LB\_IP\>/premium-asset.jpg (refresh once to observe the Age cache-hit header), then visit http\://\<LB\_IP\>/premium-asset.jpg?imop=compress,strength=medium to compress the 8.8 MB JPEG at the edge.

## **Part 2: Hands-On Exercises — Transforming `premium-asset.jpg`**

Once your Load Balancer and Cloud CDN are live, you can transform `premium-asset.jpg` on the fly at the edge by appending the `?imop=` query parameter to your Load Balancer URL. Multiple operations are chained with a semicolon (`;`), and operation attributes are separated by commas (`,`).

### **About the Master Asset (premium-asset.jpg)**

Our master workshop image (**premium-asset.jpg**, `5504×3072` pixels, **`8.8 MB`** raw JPEG) is a high-resolution studio flat-lay photograph featuring four distinct subjects arranged across a light-grey stone surface:

* **Left Subject:** Olive-green canvas travel backpack with brown leather straps (`left=0.00,right=0.68`)  
* **Top-Center Subject:** Brushed-silver camera lens (`left=0.39,right=0.40,top=0.14,bottom=0.48`) — used for rectangular product cutout cropping (`crop`)  
* **Bottom-Center Subject:** Round white navigation compass (`left=0.39,right=0.39,top=0.50,bottom=0.08`) — used for circular UI badge cropping (`crop,shape=circle`)  
* **Right Subject:** Folded topographic contour map (`left=0.75,right=0.00,top=0.18,bottom=0.22`)

### **Exercise 1: Compress Image (compress)**

Reduce the file size of the **`8.8 MB`** master JPEG at the edge while keeping its full `5504×3072` pixel dimensions using a compression preset (`very_weak`, `weak`, `medium`, `strong`, or `very_strong`):

* **URL:** `http://<LB_IP>/premium-asset.jpg?imop=compress,strength=medium`  
* **Verify:** Open **Chrome DevTools → Network**, reload the URL, and compare the compressed response size against the original `8.8 MB` image (`http://<LB_IP>/premium-asset.jpg`).

### **Exercise 2: Resize Image (resize)**

Downscale the `5504×3072` (`8.8 MB`) master image to `600 px` wide for responsive mobile and web delivery (when you specify only `width`, Cloud CDN automatically preserves the `16:9` aspect ratio at `600×335` px):

* **URL:** `http://<LB_IP>/premium-asset.jpg?imop=resize,width=600`  


### **Exercise 3: Rectangular Product Cutout — Isolating the Silver Camera Lens (crop)**

Isolate the **brushed-silver camera lens** in the upper center of `premium-asset.jpg` as a standalone square product card. In Cloud CDN Image Optimization, `left`, `right`, `top`, and `bottom` specify the fraction of the image (`0.0` to `1.0`) removed from each edge:

* **Crop Inset Fractions:** Remove `39%` from left (`left=0.39`), `40%` from right (`right=0.40`), `14%` from top (`top=0.14`), and `48%` from bottom (`bottom=0.48`), then resize to `480 px` wide and convert to `WebP`:  
* **URL:** `http://<LB_IP>/premium-asset.jpg?imop=crop,left=0.39,right=0.40,top=0.14,bottom=0.48;resize,width=480;convert,format=webp`  


### **Exercise 4: Circular UI Badge — Isolating the Navigation Compass (crop,shape=circle)**

Isolate the **round white navigation compass** in the lower center of `premium-asset.jpg` with a circular mask (`shape=circle`) and convert to `PNG` so the corners outside the circle are transparent:

* **Crop Inset Fractions:** `shape=circle`, remove `39%` from left (`left=0.39`), `39%` from right (`right=0.39`), `50%` from top (`top=0.50`), and `8%` from bottom (`bottom=0.08`), resized to `400 px` wide:  
* **URL:** `http://<LB_IP>/premium-asset.jpg?imop=crop,shape=circle,left=0.39,right=0.39,top=0.50,bottom=0.08;resize,width=400;convert,format=png`  


### **Exercise 5: Visual Filters — Blurred Hero Background (blur) & Crisp Product Detail (sharpen, tone)**

Apply edge visual filters directly from the same master file to create a soft blurred background banner and a sharpened, high-contrast product detail view:

* **Blurred Hero Background (`blur`):** `http://<LB_IP>/premium-asset.jpg?imop=resize,width=800;blur,strength=40;convert,format=webp`  
* **Crisp Product Detail (`sharpen`):** `http://<LB_IP>/premium-asset.jpg?imop=crop,left=0.39,right=0.40,top=0.14,bottom=0.48;resize,width=600;sharpen,strength=35;quality,level=85;convert,format=webp`  


### **Exercise 6: Explore Additional Transformation Types**

Experiment with other transformation types, parameter combinations, and output formats (such as `tone`, `flip`, `pad`, `quality`, and `format=avif`) directly in your browser using the official Cloud CDN Image Optimization documentation: [Supported Image Optimization Operations](https://docs.cloud.google.com/cdn/docs/image-opt-supportedops) (`https://docs.cloud.google.com/cdn/docs/image-opt-supportedops`).
