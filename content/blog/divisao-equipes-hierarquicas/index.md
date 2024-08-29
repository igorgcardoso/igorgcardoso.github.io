+++
title = "Divisão de Equipes Hierárquicas"
date = 2024-08-28
description = "Divisão de equipes hierárquicas, resolvendo o problema com grafos e Rust"
+++

Vamos ver qual é o problema e como podemos resolvê-lo com grafos.

## Problema

Em uma empresa multinacional, os funcionários têm uma hierarquia bem definida. Na empresa, há k funcionários (com IDs entre 1 e k), e cada funcionário pode ter até um chefe imediato (que nunca é ele mesmo). Um funcionário A é considerado de uma hierarquia superior a B se A é o chefe imediato de B ou se A é o chefe imediato de algum funcionário C, que está em uma hierarquia superior a B.

Na hierarquia de funcionários, na empresa não há ciclos (ou seja, não pode acontecer algo como: A ser chefe de B, B ser chefe de C e C ser chefe de A).

Para melhorar a gestão dos recursos humanos e evitar conflitos entre os funcionários, o departamento de RH decidiu dividir os funcionários em grupos de trabalho. A condição é que, em cada grupo, não devem existir dois funcionários onde um está em uma hierarquia superior ao outro.

Quantos grupos de trabalho serão necessários para abrigar todos os funcionários, seguindo a condição explicada acima?

## Entrada

A primeira linha contém um inteiro k (1 ≤ k ≤ 3000) — o número de funcionários. As próximas k linhas contêm inteiros ci (1 ≤ ci ≤ k ou ci = -1). Cada ci indica o funcionário em uma hierarquia imediatamente acima do i-ésimo funcionário. Se ci for -1, então o i-ésimo funcionário não possui nenhum chefe acima dele.

## Saída

Imprima o número de grupos de trabalho que serão necessários para abrigar todos os funcionários seguindo a condição explicada acima.

## Exemplo

Entrada

```
6
-1
1
4
1
-1
-1
```

Saída

```
3
```

## Resolução

### Modelagem

Como podemos resolver este problema com grafos? Primeiramente, vamos entender como modelar o problema.

> Em uma empresa multinacional, os funcionários têm uma hierarquia bem definida. Na empresa, há k funcionários (com IDs entre 1 e k), e cada funcionário pode ter até um chefe imediato (que nunca é ele mesmo). Um funcionário A é considerado de uma hierarquia superior a B se A é o chefe imediato de B ou se A é o chefe imediato de algum funcionário C, que está em uma hierarquia superior a B.
> Na hierarquia de funcionários não há ciclos (ou seja, não pode acontecer algo como: A ser chefe de B, B ser chefe de C e C ser chefe de A).

Levando em conta esta descrição, podemos modelar o problema como uma floresta. Onde cada árvore é uma hierarquia de funcionários. É uma floresta porque na descrição não menciona que vai ser conexo, isto é, que vai ter um superior que é chefe de todos os funcionários. Mas que haverá funcionários que não terão chefe. Deste modo, o que temos é várias árvores, podendo ser uma árvore com um único nó.

### Solução

> Para melhorar a gestão dos recursos humanos e evitar conflitos entre os funcionários, o departamento de RH decidiu dividir os funcionários em grupos de trabalho. A condição é que, em cada grupo, não devem existir dois funcionários onde um está em uma hierarquia superior ao outro.
> Quantos grupos de trabalho serão necessários para abrigar todos os funcionários, seguindo a condição explicada acima?

Tendo há modelagem, o que precisamos fazer é contar a altura das árvores. A resposta para o nosso problema é a altura da maior árvore. Porque não queremos funcionários que estão na mesma hierarquia em um mesmo grupo.

### Implementação

Vamos implementar a solução em Rust.

```rust
use std::collections::{HashMap, HashSet};

/// calcula a altura da árvore
pub fn get_height(trees: &HashMap<i32, Vec<usize>>, start: usize) -> u32 {
    let mut visited = HashSet::new();
    let mut highest_height = 0;
    let mut stack = vec![(start, 1)];
    while let Some((capybara, height)) = stack.pop() {
        if height > highest_height {
            highest_height = height;
        }
        if !visited.contains(&capybara) {
            visited.insert(capybara);
            for child in trees.get(&(capybara as i32)).unwrap_or(&Vec::new()) {
                stack.push((*child, height + 1));
            }
        }
    }

    highest_height
}

fn main() {
    // lê o número de capivaras
    let mut input = String::new();
    std::io::stdin().read_line(&mut input).unwrap();
    let num_capybaras = input.trim().parse::<usize>().unwrap();

    // lê os superiores de cada capivara
    let mut superiors = Vec::with_capacity(num_capybaras);
    let mut input = String::new();
    for _ in 0..num_capybaras {
        std::io::stdin().read_line(&mut input).unwrap();
        let capybara = input.trim().parse::<i32>().unwrap();
        superiors.push(capybara);
        input.clear();
    }

    // cria a floresta
    let mut trees: HashMap<i32, Vec<usize>> = HashMap::new();
    for (capybara, superior) in superiors.iter().enumerate() {
        trees
            .entry(*superior)
            .or_insert(Vec::new())
            .push(capybara + 1);
    }

    // calcula a altura da maior árvore
    let mut max_height = 0;
    for capybara in trees[&-1].clone() {
        let height = get_height(&trees, capybara);
        if height > max_height {
            max_height = height;
        }
    }

    println!("{}", max_height);
}
```

Após termos esse código base, podemos refatorar ele removendo um loop, que é desnecessário. Vamos fazer isso.

```rust
use std::collections::{HashMap, HashSet};

/// calcula a altura da árvore
pub fn get_height(trees: &HashMap<i32, Vec<usize>>, start: usize) -> u32 {
    let mut visited = HashSet::new();
    let mut highest_height = 0;
    let mut stack = vec![(start, 1)];
    while let Some((capybara, height)) = stack.pop() {
        if height > highest_height {
            highest_height = height;
        }
        if !visited.contains(&capybara) {
            visited.insert(capybara);
            for child in trees.get(&(capybara as i32)).unwrap_or(&Vec::new()) {
                stack.push((*child, height + 1));
            }
        }
    }

    highest_height
}

fn main() {
    // lê o número de capivaras
    let mut input = String::new();
    std::io::stdin().read_line(&mut input).unwrap();
    let num_capybaras = input.trim().parse::<usize>().unwrap();

    // cria a floresta
    // nesta versão criamos a floresta simultaneamente com a leitura
    let mut trees: HashMap<i32, Vec<usize>> = HashMap::new();
    let mut input = String::new();
    for capybara_idx in 0..num_capybaras {
        std::io::stdin().read_line(&mut input).unwrap();
        let superior = input.trim().parse::<i32>().unwrap();
        trees
            .entry(superior)
            .or_insert(Vec::new())
            .push(capybara_idx + 1);
        input.clear();
    }

    // calcula a altura da maior árvore
    let mut max_height = 0;
    for capybara in trees[&-1].clone() {
        let height = get_height(&trees, capybara);
        if height > max_height {
            max_height = height;
        }
    }

    println!("{}", max_height);
}
```
