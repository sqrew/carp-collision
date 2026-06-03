# Examples for carp-collision

## 1. Simple Sphere-Sphere Check

```carp
(load "carp-collision/collision.carp")
(use Collision)
(use CollisionChecker)

(defn main []
  (let [h1 (Handle.init 0 1)
        v1 (Volume.Ball (Sphere.init (Vector3.init 0.0 0.0 0.0) 1.0))
        c1 (Collidable.init h1 v1 1 1)
        
        h2 (Handle.init 1 1)
        v2 (Volume.Ball (Sphere.init (Vector3.init 1.5 0.0 0.0) 1.0))
        c2 (Collidable.init h2 v2 1 1)]
    
    (match (check-pair &c1 &c2)
      (Maybe.Just col) (println* "Collision! Depth: " @(Contact.depth (Collision.contact &col)))
      (Maybe.Nothing) (println* "Clear!"))))
```

## 2. Using Layers and Masks

```carp
(load "carp-collision/collision.carp")
(use Collision)
(use CollisionChecker)

(defn main []
  (let [player-layer 1
        enemy-layer 2
        projectile-layer 4
        
        ;; Projectiles hit enemies but not other projectiles
        proj (Collidable.init (Handle.init 0 1) (Volume.Ball (Sphere.init (Vector3.init 0.0 0.0 0.0) 0.1)) projectile-layer enemy-layer)
        enemy (Collidable.init (Handle.init 1 1) (Volume.Box (AABB.init (Vector3.init -1.0 -1.0 -1.0) (Vector3.init 1.0 1.0 1.0))) enemy-layer player-layer)]
    
    (match (check-pair &proj &enemy)
      (Maybe.Just _) (println* "Hit!")
      (Maybe.Nothing) (println* "No collision due to filtering or distance."))))
```

## 3. Integrating with SpatialGrid

```carp
(load "carp-collision/collision.carp")
(use Collision)
(use CollisionChecker)
(use SpatialGrid)

(defn main []
  (let [grid (SpatialGrid.new 10.0 10 10 10)
        entities [(Collidable.init (Handle.init 0 1) (Volume.Ball (Sphere.init (Vector3.init 5.0 5.0 5.0) 1.0)) 1 1)
                  (Collidable.init (Handle.init 1 1) (Volume.Ball (Sphere.init (Vector3.init 6.0 5.0 5.0) 1.0)) 1 1)]]
    (do
      ;; Sync grid with entities
      (for [i 0 (Array.length &entities)]
        (let [e (Array.unsafe-nth &entities i)]
          (SpatialGrid.insert! &grid &(Collision.get-aabb (Collidable.volume e)) (the Uint64 (Int.to-uint64 i)))))
      
      ;; Query collisions for the first entity
      (let [self (Array.unsafe-nth &entities 0)
            hits (query-and-check &grid self &entities)]
        (println* "Entity 0 hit " (Array.length &hits) " others.")))))
```
