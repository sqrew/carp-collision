# carp-collision

A modular 3D collision detection and dispatch library for the [Carp](https://github.com/carp-lang/Carp) programming language.

This library bridges the gap between spatial partitioning (`carp-spatial`) and geometric intersection primitives (`carp-geometry`). It provides a high-level API for narrow-phase collision checks, filtering, and contact manifold generation.

## Features
- **Narrow-phase Dispatch**: Automatically chooses the correct `geometry` test for `Box` (AABB) and `Ball` (Sphere) volumes.
- **Collision Filtering**: Bitmask-based `layer` and `mask` system to ignore irrelevant collisions.
- **Contact Data**: Returns depth, point, and normals for physical resolution.
- **Spatial Integration**: Designed to work seamlessly with `SpatialGrid` for $O(n \log n)$ performance.

## Usage

```carp
(load "carp-collision/collision.carp")
(use Collision)
(use CollisionChecker)

;; 1. Define Collidables
(let [h1 (Handle.init 0 1)
      v1 (Volume.Ball (Sphere.init (Vector3.init 0.0 0.0 0.0) 1.0))
      c1 (Collidable.init h1 v1 1 1)
      
      h2 (Handle.init 1 1)
      v2 (Volume.Ball (Sphere.init (Vector3.init 1.5 0.0 0.0) 1.0))
      c2 (Collidable.init h2 v2 1 1)]
  
  ;; 2. Check for collision
  (match (check-pair &c1 &c2)
    (Maybe.Just col) (println* "Collision detected! Depth: " @(Contact.depth (Collision.contact &col)))
    (Maybe.Nothing) (println* "No collision.")))
```

## Integration with carp-spatial
The `CollisionChecker.query-and-check` function takes a `SpatialGrid` and a list of entities to perform efficient broad-phase filtered queries.

## License
MIT
