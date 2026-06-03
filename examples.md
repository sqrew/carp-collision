# Examples for carp-collision

## 1. Simple Sphere-Sphere Check

```carp
(load "carp-collision/collision.carp")
(use Collision)
(use CollisionChecker)

(defn main []
  (let [h1 (Handle.init (Uint64.from-long 0l) (Uint32.from-long 1l))
        v1 (Volume.Ball (Sphere.init (Vector3.init 0.0 0.0 0.0) 1.0))
        c1 (Collidable.init h1 v1 (Uint32.from-long 1l) (Uint32.from-long 1l) false)
        
        h2 (Handle.init (Uint64.from-long 1l) (Uint32.from-long 1l))
        v2 (Volume.Ball (Sphere.init (Vector3.init 1.5 0.0 0.0) 1.0))
        c2 (Collidable.init h2 v2 (Uint32.from-long 1l) (Uint32.from-long 1l) false)]
    
    (match (check-pair &c1 &c2)
      (Maybe.Just (CollisionResult.Physical col)) 
          (println* "Collision! Depth: " @(Contact.depth (Collision.contact &col)))
      _ (println* "No physical collision."))))
```

## 2. Using Layers and Masks (Symmetric)

```carp
(load "carp-collision/collision.carp")
(use Collision)
(use CollisionChecker)

(defn main []
  (let [player-layer (Uint32.from-long 1l)
        enemy-layer (Uint32.from-long 2l)
        projectile-layer (Uint32.from-long 4l)
        
        ;; Projectiles hit enemies
        proj (Collidable.init (Handle.init (Uint64.from-long 0l) (Uint32.from-long 1l)) 
                             (Volume.Ball (Sphere.init (Vector3.init 0.0 0.0 0.0) 0.1)) 
                             projectile-layer enemy-layer false)
        ;; Enemies hit players
        enemy (Collidable.init (Handle.init (Uint64.from-long 1l) (Uint32.from-long 1l)) 
                              (Volume.Box (AABB.init (Vector3.init -1.0 -1.0 -1.0) (Vector3.init 1.0 1.0 1.0))) 
                              enemy-layer player-layer false)]
    
    ;; This will result in Nothing because filtering is symmetric:
    ;; proj.mask (enemy) matches enemy.layer (enemy) -> YES
    ;; BUT enemy.mask (player) DOES NOT match proj.layer (projectile) -> NO
    (match (check-pair &proj &enemy)
      (Maybe.Just _) (println* "Hit!")
      (Maybe.Nothing) (println* "Symmetric filter blocked collision."))))
```

## 3. Integrating with SpatialGrid and Avoiding Duplicates

```carp
(load "carp-collision/collision.carp")
(use Collision)
(use CollisionChecker)
(use SpatialGrid)

(defn main []
  (let [grid (SpatialGrid.new 10.0 10 10 10)
        entities [(Collidable.init (Handle.init (Uint64.from-long 0l) (Uint32.from-long 1l)) 
                                  (Volume.Ball (Sphere.init (Vector3.init 5.0 5.0 5.0) 1.0)) 
                                  (Uint32.from-long 1l) (Uint32.from-long 1l) false)
                  (Collidable.init (Handle.init (Uint64.from-long 1l) (Uint32.from-long 1l)) 
                                  (Volume.Ball (Sphere.init (Vector3.init 6.0 5.0 5.0) 1.0)) 
                                  (Uint32.from-long 1l) (Uint32.from-long 1l) false)]]
    (do
      ;; Sync grid with entities
      (for [i 0 (Array.length &entities)]
        (let [e (Array.unsafe-nth &entities i)]
          (SpatialGrid.insert! &grid &(Collision.get-aabb (Collidable.volume e)) (Uint64.from-long (Int.to-long i)))))
      
      ;; Query collisions for the first entity, using symmetric flag to avoid duplicate pairs in broad loops
      (let [self (Array.unsafe-nth &entities 0)
            hits (query-and-check &grid self &entities true)]
        (println* "Entity 0 hit " (Array.length &hits) " others.")))))
```
