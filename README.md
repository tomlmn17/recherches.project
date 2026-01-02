# recherches.project
Les Algorithmes gloutons pour le robot médical de livraison
1 | Principe des algorithmes gloutons

Un algorithme glouton construit une solution étape par étape, en faisant à chaque étape le choix localement optimal, sans revenir en arrière.
Dans notre projet :

A chaque position du robot, on choisit le service le plus proche non encore livré, jusqu’à ce que tous les services soient visités.
Ce type d’algorithme : est très rapide, mais ne garantit pas l’optimalité globale (contrairement à la recherche exhaustive).

Problème traité : variante du TSP
On modélise le problème comme suit :

Un graphe pondéré 𝐺=(𝑉,𝐸)=(V,E)
V : services de l’hôpital
E : déplacements possibles
poids = distance ou temps
point de départ fixe (ex. pharmacie)

Objectif :
--> visiter tous les sommets une seule fois en minimisant la distance totale
Algorithme glouton du plus proche voisin : Principe

Le robot commence au point de départ.
Il cherche le service non visité le plus proche.
Il s’y rend et marque ce service comme visité.
On répète jusqu’à ce que tous les services soient livrés.


Modélisation par matrice d’adjacence v1 :
On utilise une matrice carrée M de taille n × n :

M[i][j] = distance entre le service i et le service j
M[i][i] = 0
si aucun lien direct : valeur très grande (∞)

      0   1   2   3
0   [ 0   4   2   7 ]
1   [ 4   0   5   1 ]
2   [ 2   5   0   6 ]
3   [ 7   1   6   0 ]

Entrée :
- M : matrice d’adjacence de taille n × n
- depart : sommet de départ

Sortie :
- chemin : ordre de visite des services
- distance_totale

Initialisation :
- courant ← depart
- visite[0..n-1] ← faux
- visite[courant] ← vrai
- chemin ← [courant]
- distance_totale ← 0

Tant que tous les sommets ne sont pas visités :
    min_distance ← +∞
    prochain ← -1

    Pour chaque sommet j de 0 à n-1 :
        Si visite[j] = faux ET M[courant][j] < min_distance :
            min_distance ← M[courant][j]
            prochain ← j

    distance_totale ← distance_totale + min_distance
    courant ← prochain
    visite[courant] ← vrai
    ajouter courant à chemin

Retourner chemin et distance_totale

Modélisation sans matrice d’adjacence v1 :

0 → (1,4), (2,2), (3,7)
1 → (0,4), (2,5), (3,1)
2 → (0,2), (1,5), (3,6)
3 → (0,7), (1,1), (2,6)

Entrée :
- adj : liste d’adjacence
- depart : sommet de départ

Initialisation :
- courant ← depart
- visite ← ensemble vide
- visite ← visite ∪ {courant}
- chemin ← [courant]
- distance_totale ← 0

Tant que |visite| < nombre de sommets :
    min_distance ← +∞
    prochain ← -1

    Pour chaque (voisin, distance) dans adj[courant] :
        Si voisin ∉ visite ET distance < min_distance :
            min_distance ← distance
            prochain ← voisin

    distance_totale ← distance_totale + min_distance
    courant ← prochain
    visite ← visite ∪ {courant}
    ajouter courant à chemin

Retourner chemin et distance_totale



Procédons à l'implémentation en C et Ocaml avec matrices d'ajacences : Matrice dist[n][n] | Tableau visited | Choix glouton du plus proche voisin :
Code en C :

#include <stdio.h>
#include <stdbool.h>
#include <limits.h>

#define N 4   // nombre de services
#define INF 1000000

void nearest_neighbor(int dist[N][N], int start) {
    bool visited[N] = {false};
    int current = start;
    int total_distance = 0;

    visited[current] = true;
    printf("Chemin : %d ", current);

    for (int step = 1; step < N; step++) {
        int next = -1;
        int min_dist = INF;

        for (int i = 0; i < N; i++) {
            if (!visited[i] && dist[current][i] < min_dist) {
                min_dist = dist[current][i];
                next = i;
            }
        }

        visited[next] = true;
        total_distance += min_dist;
        current = next;

        printf("-> %d ", current);
    }

    printf("\nDistance totale : %d\n", total_distance);
}

int main() {
    int dist[N][N] = {
        {0, 4, 2, 7},
        {4, 0, 5, 1},
        {2, 5, 0, 6},
        {7, 1, 6, 0}
    };

    nearest_neighbor(dist, 0);
    return 0;
}


Code en Ocaml :

let inf = 1_000_000

let nearest_neighbor dist start =
  let n = Array.length dist in
  let visited = Array.make n false in
  let current = ref start in
  let total = ref 0 in

  visited.(start) <- true;
  Printf.printf "Chemin : %d " start;

  for _ = 1 to n - 1 do
    let min_dist = ref inf in
    let next = ref (-1) in

    for i = 0 to n - 1 do
      if not visited.(i) && dist.(!current).(i) < !min_dist then begin
        min_dist := dist.(!current).(i);
        next := i
      end
    done;

    visited.(!next) <- true;
    total := !total + !min_dist;
    current := !next;

    Printf.printf "-> %d " !current
  done;

  Printf.printf "\nDistance totale : %d\n" !total

let () =
  let dist = [|
    [|0; 4; 2; 7|];
    [|4; 0; 5; 1|];
    [|2; 5; 0; 6|];
    [|7; 1; 6; 0|]
  |] in
  nearest_neighbor dist 0


Procédons maintenant avec une liste d'adjacence et non des matrices d'adjacences : 
Code en C : 

#include <stdio.h>
#include <stdbool.h>
#include <limits.h>

#define N 4
#define MAX_NEIGHBORS 4
#define INF 1000000

typedef struct {
    int to;
    int weight;
} Edge;

typedef struct {
    Edge neighbors[MAX_NEIGHBORS];
    int count;
} Node;

void nearest_neighbor(Node graph[N], int start) {
    bool visited[N] = {false};
    int current = start;
    int total_distance = 0;

    visited[current] = true;
    printf("Chemin : %d ", current);

    for (int step = 1; step < N; step++) {
        int next = -1;
        int min_dist = INF;

        for (int i = 0; i < graph[current].count; i++) {
            int v = graph[current].neighbors[i].to;
            int d = graph[current].neighbors[i].weight;

            if (!visited[v] && d < min_dist) {
                min_dist = d;
                next = v;
            }
        }

        visited[next] = true;
        total_distance += min_dist;
        current = next;

        printf("-> %d ", current);
    }

    printf("\nDistance totale : %d\n", total_distance);
}

int main() {
    Node graph[N] = {
        {{{1,4},{2,2},{3,7}}, 3},
        {{{0,4},{2,5},{3,1}}, 3},
        {{{0,2},{1,5},{3,6}}, 3},
        {{{0,7},{1,1},{2,6}}, 3}
    };

    nearest_neighbor(graph, 0);
    return 0;
}


Code en Ocaml :

let inf = 1_000_000

let nearest_neighbor graph start =
  let n = Array.length graph in
  let visited = Array.make n false in
  let current = ref start in
  let total = ref 0 in

  visited.(start) <- true;
  Printf.printf "Chemin : %d " start;

  for _ = 1 to n - 1 do
    let min_dist = ref inf in
    let next = ref (-1) in

    List.iter (fun (v, d) ->
      if not visited.(v) && d < !min_dist then begin
        min_dist := d;
        next := v
      end
    ) graph.(!current);

    visited.(!next) <- true;
    total := !total + !min_dist;
    current := !next;

    Printf.printf "-> %d " !current
  done;

  Printf.printf "\nDistance totale : %d\n" !total

let () =
  let graph = [|
    [(1,4); (2,2); (3,7)];
    [(0,4); (2,5); (3,1)];
    [(0,2); (1,5); (3,6)];
    [(0,7); (1,1); (2,6)]
  |] in
  nearest_neighbor graph 0


CONCLUSION : Les algorithmes gloutons, bien que non optimaux, offrent une solution rapide et simple pour le pilotage du robot médical.L’utilisation d’une matrice d’adjacence facilite l’implémentation mais devient coûteuse en mémoire, tandis que la liste d’adjacence permet une meilleure adaptation aux graphes de grande taille, plus proches d’un environnement hospitalier réel.

















