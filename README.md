# carp-collision

A modular 3D collision detection and dispatch library for the [Carp](https://github.com/carp-lang/Carp) programming language.

This library bridges the gap between spatial partitioning (`carp-spatial`) and geometric intersection primitives (`carp-geometry`). It provides a high-level API for narrow-phase collision checks, filtering, and contact manifold generation.

## Features
- **Narrow-phase Dispatch**: Automatically chooses the correct `geometry` test for `Box` (AABB) and `Ball` (Sphere) volumes.
- **Collision Filtering**: Bitmask-based `layer` and `mask` system to ignore irrelevant collisions.
- **Contact Data**: Returns depth, point, and normals for physical resolution.
- **Spatial Integration**: Designed to work seamlessly with `SpatialGrid` for $O(n \log n)$ performance.


## Examples

See [examples.md](examples.md) for usage examples.
## Integration with carp-spatial
The `CollisionChecker.query-and-check` function takes a `SpatialGrid` and a list of entities to perform efficient broad-phase filtered queries.

## License
MIT
