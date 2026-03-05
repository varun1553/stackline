# Stackline Full Stack Assignment – Bug Fixes & Improvements

## Overview

This project is a sample eCommerce web application that includes:

1. Product List Page

2. Search Results Page

3. Product Detail Page

While reviewing the codebase, several issues were identified across UI components, API integration, routing logic, and data handling. These issues included UX problems, runtime crashes, filtering inconsistencies, and potential security concerns.

The objective of this task was to:

1. Identify bugs in the application

2. Fix them with minimal disruption to the existing structure

3. Improve stability and reliability

4. Clearly document the reasoning behind each fix

This document explains the issues found and the solutions implemented.

# Issues Identified and Fixes Implemented

## Button Variant Class Merging Bug

### Issue

In button.tsx, the Button component incorrectly passed className inside the cva() variant configuration:

    className={cn(buttonVariants({ variant, size, className }))}

The cva() utility expects only variant configuration properties, not arbitrary class names.

Passing className into the configuration can lead to:

- Incorrect Tailwind class merging

- Unexpected style overrides

- Inconsistent styling across variants

### Fix

className was removed from the cva() configuration and merged separately.

    className={cn(buttonVariants({ variant, size }), className)}
 
### Why This Fix

Separating variant configuration from additional classes ensures:

- Proper Tailwind class merging

- Predictable styling behavior

- Better compatibility with component variants
## Input Component Missing Default Type

### Issue

The Input component accepted a type prop but did not define a default value.

    function Input({ className, type, ...props })

If type is undefined, browsers may handle the input type inconsistently.

### Fix

A default value was added.
    
    <input type={type ?? "text"}

### Why This Fix

Providing a default type ensures:

- Consistent browser behavior

- Predictable input rendering

- More reliable form handling


## Subcategory API Not Using Selected Category

### Issue

The application fetched subcategories using:

    fetch(`/api/subcategories`)

The selected category was not passed to the API.

This resulted in:

Incorrect subcategory results

Broken category → subcategory filtering logic

### Fix

The selected category is now included in the request.

    fetch(`/api/subcategories?category=${selectedCategory}`)

### Why This Fix

Including the selected category ensures:

- Accurate subcategory results

- Correct filter behavior

- Proper relationship between categories and subcategories

## Missing Error Handling for API Requests

### Issue

Many API calls were written as:

    fetch(...).then(res => res.json())

This assumes that every response is successful and contains valid JSON.

If the API returns:

- HTTP errors (500, 404)

- Invalid JSON

- Network failures

the application could crash.

### Fix

Error handling was added to safely handle failed requests.

       fetch(`/api/products?${params}`)
         .then((res) => {
           if (!res.ok) throw new Error("API error");
           return res.json();
         })
         .then((data) => setProducts(data.products || []))
         .catch((err) => {
           console.error(err);
           setProducts([]);
         })
         .finally(() => setLoading(false));
  
### Why This Fix

This prevents:

- Application crashes

- Broken UI states

- Poor user experience when APIs fail


## Possible Image Rendering Crash

### Issue

Product cards used:

    product.imageUrls[0]

If imageUrls is undefined or empty, this causes a runtime error.

### Fix

Optional chaining and fallback logic were added.

    src={product.imageUrls?.[0] || "/placeholder.png"}

### Why This Fix

This ensures:

- UI stability even when image data is missing

- Graceful fallback images

- Prevention of rendering crashes

## Product Data Passed Through URL (Routing Issue)

### Issue

The application passed full product objects through the router:

    query: { product: JSON.stringify(product) }

This creates several problems:

- URLs become very large

- Sensitive product data becomes exposed

- Users can manipulate product information

- Potential routing instability

### Fix

Only the product SKU is passed through the URL.

    query: { sku: product.stacklineSku }

The product page then fetches product details using the SKU.

### Why This Fix

This improves:

- Security

- URL readability

- Data integrity

- Alignment with common API-based architectures


## Image Loading Error with Next.js External Domains

### Issue

While loading product images, the application produced this error:

    Invalid src prop (...) on next/image, hostname is not configured

This occurred because product images were served from Amazon image domains that were not allowed in the Next.js configuration.

Example domains:

    m.media-amazon.com
    images-na.ssl-images-amazon.com
    
### Fix

The domains were added to next.config.ts.

    
          {
            protocol: "https",
            hostname: "images-na.ssl-images-amazon.com",
          },
        

### Why This Fix

Next.js requires external domains to be explicitly allowed for:

- Security

- Image optimization

- Proper rendering of remote images

## Incorrect Product Results When Changing Category Filter

### Issue

Changing categories sometimes displayed "No products found", even though products existed.

Example scenario:

Category: Electronics

Subcategory: Laptop → products load correctly

User switches to Books

UI shows No products found

Root Cause

The previously selected subcategory was still being sent in the API request.

Example request:

    /api/products?category=Books&subCategory=Laptop

Since Laptop is not a valid subcategory for Books, the API returned an empty result.

### Fix

The subcategory state is now reset whenever the category changes.

    <Select
      value={selectedCategory}
      onValueChange={(value) => {
        setSelectedCategory(value);
        setSelectedSubCategory(undefined);
      }}
    >
    
### Result

After this fix:

- Subcategory resets automatically

- API requests always contain valid filters

- Products display correctly for the selected category

Impact

This improves:

- Filter reliability

- API request accuracy

- Overall user experience

## Summary

During this review, several improvements were implemented across the application to address:

UI component bugs

API request handling

Filtering logic

Search performance

Image loading configuration

Routing and data security

These fixes significantly improve:

Application stability

User experience

Code reliability

Maintainability of the codebase
