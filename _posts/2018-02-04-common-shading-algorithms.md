---
layout: post
title: "common shading algorithms"
description: "common shading algorithms"
category: CG
tags: [CG]
---

&nbsp; &nbsp; &nbsp; &nbsp;最近一周发现，自己的水平已经弱到给大佬们打下手都不配了，心情非常紧张。趁着春节前最后一个周末，复(yu)习一下shader相关的一些东西。

<!-- more -->

* Table of Contents
{:toc}

# Interpolation techniques

When calculating the brightness of a surface during rendering, our illumination model requires that we know the surface normal. However, a 3D model is usually described by a polygon mesh , which may only store the surface normal at a limited number of points, usually either in the vertices, in the polygon faces, or in both. To get around this problem, one of a number of interpolation techniques can be used.

## Flat shading

Here, a color is calculated for one point on each polygon (usually for the first vertex in the polygon, but sometimes for the centroid for triangle meshes), based on the polygon's surface normal and on the assumption that all polygons are flat. The color everywhere else is then interpolated by coloring all points on a polygon the same as the point for which the color was calculated, giving each polygon a uniform color (similar to in [nearest-neighbor interpolation](https://en.wikipedia.org/wiki/Nearest-neighbor_interpolation)). It is usually used for high speed rendering where more advanced shading techniques are too computationally expensive. As a result of flat shading all of the polygon's vertices are colored with one color, allowing differentiation between adjacent polygons. Specular highlights are rendered poorly with flat shading: If there happens to be a large specular component at the representative vertex, that brightness is drawn uniformly over the entire face. If a specular highlight doesn’t fall on the representative point, it is missed entirely. Consequently, the specular reflection component is usually not included in flat shading computation.

## Smooth shading

In contrast to flat shading where the colors change discontinuously at polygon borders, with smooth shading the color changes from pixel to pixel, resulting in a smooth color transition between two adjacent polygons. Usually, values are first calculated in the vertices and bilinear interpolation is then used to calculate the values of pixels between the vertices of the polygons.

### Gouraud shading

Gouraud shading, named after Henri Gouraud, is an interpolation method used in computer graphics to produce continuous shading of surfaces represented by polygon meshes. In practice, Gouraud shading is most often used to achieve continuous lighting on triangle surfaces by computing the lighting at the corners of each triangle and linearly interpolating the resulting colours for each pixel covered by the triangle. Gouraud first published the technique in 1971.

Gouraud shading works as follows: An estimate to the surface normal of each vertex in a polygonal 3D model is either specified for each vertex or found by averaging the surface normals of the polygons that meet at each vertex. Using these estimates, lighting computations based on a reflection model, e.g. the Phong reflection model, are then performed to produce colour intensities at the vertices. For each screen pixel that is covered by the polygonal mesh, colour intensities can then be interpolated from the colour values calculated at the vertices.

### Phong shading

Phong shading improves upon Gouraud shading and provides a better approximation of the shading of a smooth surface. Phong shading assumes a smoothly varying surface normal vector. The Phong interpolation method works better than Gouraud shading when applied to a reflection model that has small specular highlights such as the Phong reflection model.

The most serious problem with Gouraud shading occurs when specular highlights are found in the middle of a large polygon. Since these specular highlights are absent from the polygon's vertices and Gouraud shading interpolates based on the vertex colors, the specular highlight will be missing from the polygon's interior. This problem is fixed by Phong shading.

Unlike Gouraud shading, which interpolates colors across polygons, in Phong shading a normal vector is linearly interpolated across the surface of the polygon from the polygon's vertex normals. The surface normal is interpolated and normalized at each pixel and then used in a reflection model, e.g. the Phong reflection model, to obtain the final pixel color. Phong shading is more computationally expensive than Gouraud shading since the reflection model must be computed at each pixel instead of at each vertex.

In modern graphics hardware, variants of this algorithm are implemented using pixel or fragment shaders.

# Illumination models

## Realistic

The illumination models listed here attempt to model the perceived brightness of a surface or a component of the brightness in a way that looks realistic. Some take physical aspects into consideration, like for example the [Fresnel equations](https://en.wikipedia.org/wiki/Fresnel_equations), [microfacets](https://en.wikipedia.org/wiki/Specular_highlight#Microfacets), the [rendering equation](https://en.wikipedia.org/wiki/Rendering_equation) and [subsurface scattering](https://en.wikipedia.org/wiki/Subsurface_scattering).

### Diffuse reflection

Light that is reflected on a non-metallic and/or a very rough surface gives rise to a diffuse reflection. Models that describe the perceived brightness due to diffuse reflection include:

1. [Lambert](https://en.wikipedia.org/wiki/Lambertian_reflectance)
2. [Oren–Nayar](https://en.wikipedia.org/wiki/Oren%E2%80%93Nayar_reflectance_model) (Rough opaque diffuse surfaces)
3. [Minnaert](https://en.wikipedia.org/wiki/Minnaert_function)

### Specular reflection

Light that is reflected on a relatively smooth surface gives rise to a specular reflection. This kind of reflection is especially strong for metal surfaces. Models that describe the perceived brightness due to specular reflection include:

1. [Phong](https://en.wikipedia.org/wiki/Phong_reflection_model)
2. [Blinn–Phong](https://en.wikipedia.org/wiki/Blinn%E2%80%93Phong_shading_model)
3. [Cook–Torrance](https://en.wikipedia.org/wiki/Specular_highlight#Cook%E2%80%93Torrance_model) (microfacets)
4. [Ward anisotropic](https://en.wikipedia.org/wiki/Specular_highlight#Ward_anisotropic_distribution)

### Subsurface scattering

Subsurface scattering is an indirect form of reflection where some of the light is transmitted into a semi-transparent material, scattered under the surface and bounced back out again. The light that is not absorbed by the material and bounced out through the surface again gives rise to a diffuse indirect reflection, which will illuminate the surface not only where it is lit, but also in the vicinity of where the light hits, as well as on the other side of thin parts of an object. Most non-metals can transmit light to a certain degree and are therefore affected by this effect. Subsurface scattering models include:

- [Hanrahan–Krueger](https://en.wikipedia.org/w/index.php?title=Hanrahan%E2%80%93Krueger&action=edit&redlink=1) model of subsurface scattering

## Non-photorealistic

Non-photorealistic illumination models don't attempt to model the perceived brightness of a surface in a realistic way, but focuses expressing certain styles. They are used for example in cartoons, video games, movies or technical illustrations, and include:

1. [Cel shading](https://en.wikipedia.org/wiki/Cel_shading)
2. [Gooch shading](https://en.wikipedia.org/wiki/Gooch_shading)

# Shading Models samples

## Lambert

![Lambert](http://softimage.wiki.softimage.com/xsidocs/fi099e45.jpg)

Uses the ambient and diffuse colors to create a matte surface with no specular highlights. It interpolates between normals of adjacent surface triangles so that the shading changes progressively, creating a matte surface.

The result is a smoothly shaded object, like an egg or a ping-pong ball. Reflectivity, transparency, refraction, and texture can be applied to an object shaded with a Lambert shader.

## Phong

![Phong](http://softimage.wiki.softimage.com/xsidocs/fi099e3b.jpg)

Uses ambient, diffuse, and specular colors. This shading model reads the surface normals’ orientation and interpolates between them to create an appearance of smooth shading. It also processes the relation between normals, the light, and the camera’s point of view to create a specular highlight.

The result is a smoothly shaded object with diffuse and ambient areas of illumination on its surface and a specular highlight so that the object appears shiny, like a billiard ball or plastic. Reflectivity, transparency, refraction, and texture can be applied to an object shaded with a Phong shader.

## Blinn-Phong

![Blinn](http://softimage.wiki.softimage.com/xsidocs/fi099e4f.jpg)


Uses diffuse, ambient, and specular color, as well as a refractive index for calculating the specular highlight. This shading model produces results that are virtually identical to the Phong shading model except that the shape of the specular highlight reflects the actual lighting more accurately when there is a high angle of incidence between the camera and the light.

This shading model is useful for rough or sharp edges and simulating a metal surface. The specular highlight also appears brighter than the Phong model. Reflectivity, transparency, refraction, and texture can be applied to an object shaded with a Blinn shader.

## Cook-Torrance

![Cook-Torrance](http://softimage.wiki.softimage.com/xsidocs/fi099e59.jpg)

Uses diffuse, ambient, and specular color, as well as a refractive index used to calculate the specular highlight. It reads the surface normals’ orientation and interpolates between them to create an appearance of smooth shading. It also processes the relation between normals, the light, and the camera’s point of view to create a specular highlight.

This shading model produces results that are somewhere between the Blinn and Lambert shading models and is useful for simulating smooth and reflective objects, such as leather. Reflectivity, transparency, refraction, and texture can be applied to an object shaded with a Cook-Torrance shader.

Because this shading model is more complex to calculate, it takes longer to render than the other shading models.

## Strauss

![Strauss](http://softimage.wiki.softimage.com/xsidocs/fi099e64.jpg)

Uses only the diffuse color to simulate a metal surface. The surface’s specular is defined with smoothness and “metalness” parameters that control the diffuse to specular ratio as well as reflectivity and highlights.

Reflectivity, transparency, refraction, and texture can be applied to an object shaded with a Strauss shader.

## Anisotropic

![Anisotropic](http://softimage.wiki.softimage.com/xsidocs/fi099e6e.jpg)

Sometimes called Ward, this shading model simulates a glossy surface using an ambient, diffuse, and a glossy color. To create a “brushed” effect, such as brushed aluminum, it is possible to define the specular color’s orientation based on the object’s surface orientation. The specular is calculated using UV coordinates.

Reflectivity, transparency, refraction, and texture can be applied to an object shaded with an anisotropic shader.

# References

1. [wikipedia](https://en.wikipedia.org/wiki/List_of_common_shading_algorithms)
2. [softimage wiki](http://softimage.wiki.softimage.com/xsidocs/surface_shaders_ShadingModels.htm)