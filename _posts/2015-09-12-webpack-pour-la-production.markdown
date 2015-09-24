---
layout: post
title:  "Webpack pour la production"
date:   2015-09-12 11:58:42
description: Mise en place d'un configuration de production avec Webpack
categories:
- webpack
- react
permalink: webpack-pour-la-production
---

Dans le [précédent article][precedent-article], on a vu comment mettre en place une stack de développement autour de React et Webpack. Parce que vous etes (surement) entrain de lancer le prochain Facebook, il va falloir créer la configuration adéquate pour packager l'appli en mode production. <br><br>Au programme : minification des assets, optimisations diverses et désactivation des sources-map et du débug.

Pour s'aider, on va repartir du tuto précédant et réutiliser le **webpack.config.js** disponible sur ce [gist][gist]. Ce dernier correspond à une configuration Webpack de développement. L'idée c'est de conserver ce fichier pour le mode développement et d'en créer un second pour le mode production.

Bien sûr, ces deux configs vont partager un certain nombre de propriétés en commun. Pour être *DRY* (Don't Repeat Yourself), on va externaliser ces propriétés dans un fichier de config de base. Puis on ajoutera dans chaque config (dev et prod) leurs propriétés spécifiques ayant au préalable inclus la config de base.

Voici l'arborescence corespondantes :

    - webpack.base.config.js
    - webpack.dev.config.js
    - webpack.prod.config.js

#### Configuration de base
C'est ici que l'on va réunir toutes les propriétés communes à nos deux configurations.

Ce qui nous donne :
{% highlight javascript %}
var path = require("path");
var webpack = require("webpack");

module.exports = {
    entry: ['./app/index.js'],
    output: {
        path: path.join(__dirname, './build'),
        filename: 'bundle.js',
        publicPath: '/'
    },
    resolve: {
        extensions: ['', '.js', '.jsx']
    },
    module: {
        preLoaders: [
            {
                test: /\.jsx?$/,
                loader: 'eslint-loader',
                exclude: /node_modules/
            }
        ],
        loaders: [
            {
                test: /\.jsx?$/,
                exclude: /node_modules/,
                loader: 'babel?optional[]=runtime'
            },
            {
                test: /\.(png|jpg|woff|woff2|eot|ttf|svg)$/,
                loader: 'url-loader?limit=100000'
            }
        ]
    }
};
{% endhighlight %}

#### Configuration de développement
Pour fusionner nos configs Webpack on va utiliser [lodash][lodash], une lib bien pratique qui permet d'utiliser de nombreuses méthodes utilitaires.

On l'installe via npm :
{% highlight console %}
$ npm install lodash --save
{% endhighlight %}

On oublie pas de faire un require de lodash et d'inclure également notre config de base. Vient ensuite le code spécifique au mode dev. Rien de nouveau ici c'est le même code qu'il y avait dans le gist de départ mais déplacé dans le fichier adéquat. Ensuite, on utilise la méthode `merge` de lodash pour fusionner cette config avec celle de base. On passe à cette méthode une fonction pour préciser que les valeurs des propriétés de type array devront être concaténées (plutôt que remplacées). Enfin on exporte notre config.

{% highlight javascript %}
var baseConfig = require('./webpack.config.js');
var _ = require('lodash');
var ExtractTextPlugin = require('extract-text-webpack-plugin');

var devConfig = {
    devtool: 'source-map',
    module: {
        loaders: [{
            test: /\.scss$/,
            loader: ExtractTextPlugin.extract(
                'css?sourceMap!' +
                'sass?sourceMap'
            )
        }]
    },
    plugins: [
        new ExtractTextPlugin('styles.css')
    ]
};

// Merge base config with dev config
var devConfig = _.merge(baseConfig, devConfig, function(a, b) {
    if (_.isArray(a)) {
        return a.concat(b);
    }
});

module.exports = devConfig;
{% endhighlight %}

#### Configuration de production
Même principe que pour la configuration de développement. On inclut la config de base et on enrichit/surcharge avec les propriétés spécifiques à la production.

On commence par se passer du plugin `ExtractTextPlugin` qui permettait d'extraire les CSS dans une feuille de styles dédiée et on supprime également l'option `devtool`.

A l'inverse, on ajoute le plugin `UglifyJsPlugin` embarqué nativement par Webpack. Ce dernier permet de minifier nos scripts et nos feuilles de styles. Avec un environnement React, ce plugin génère quelques warnings superflus, on va donc les désactivés.

Second plugin utile pour la production, `DedupePlugin`, en charge d'éliminer les doublons lorsque des libs utilisent des dépendances communes.

{% highlight javascript %}
var baseConfig = require('./webpack.base.config.js');
var webpack = require("webpack");
var _ = require('lodash');

var prodConfig = {
    module: {
        loaders: [{
            test: /\.scss$/,
            loader: 'style!css!sass'
        }]
    },
    plugins: [
        new webpack.optimize.UglifyJsPlugin({
            minimize: true,
            compress: {
                warnings: false
            }
        }),
        new webpack.optimize.DedupePlugin()
    ]
};

// Merge the base config with the prod config
var prodConfig = _.merge(baseConfig, prodConfig, function(a, b) {
    if (_.isArray(a)) {
        return a.concat(b);
    }
});

module.exports = prodConfig;
{% endhighlight %}

#### package.json
Afin de lancer nos nouvelles configurations, le **package.json** va être légerement modifié. On va se servir de l'option `--config` de Webpack qui permet d'indiquer un fichier différent du traditionnel **webpack.config.js**.

Pour le mode de prodution on set la variable `NODE_ENV` à *production* pour permettre à nos modules et plugins d'agir en conséquence en réalisant diverses optimisations. Idem pour l'option `-p` que l'on passe à Webpack (plus d'infos [ici][ici1] et [ici][ici2]).

{% highlight javascript %}
"scripts": {
  "dev": "webpack-dev-server --port 8080 --hot --inline --config webpack.dev.config.js",
  "deploy": "NODE_ENV=production webpack -p --config webpack.production.config.js"
}
{% endhighlight %}

## Lancement et résultats
#### Dev
Pour lancer notre config de dev :
{% highlight console %}
$ npm run dev
{% endhighlight %}

Résultat du build :
{% highlight console %}
Asset       Size  Chunks             Chunk Names
bundle.js     859 kB       0  [emitted]  main
styles.css   62 bytes       0  [emitted]  main
bundle.js.map    1.03 MB       0  [emitted]  main
styles.css.map  392 bytes       0  [emitted]  main
{% endhighlight %}

On obtient bien notre fichier **bundle.js** (859 kB) non minifié et le source-maps associé **bundle.js.map** pour le debug. Même chose pour les CSS regroupés dans le fichier **styles.css** et son source-maps **styles.css.map**. L'appli est toujours disponible sur [http://localhost:8080][localhost].

#### Prod
La config de prod se lance de la façon suivante :
{% highlight console %}
$ npm run deploy
{% endhighlight %}

Résultat du build :
{% highlight console %}
Asset    Size  Chunks             Chunk Names
bundle.js  175 kB       0  [emitted]  main
{% endhighlight %}

Côté prodution, plus aucun source-maps ni de feuille de styles dédiée, enfin un **bundle.js** minifié qui ne pèse plus que 175kB. Simple et efficace. Il ne reste plus qu'à servir ces ressources statiques du répertoire **build** par un serveur web type Apache ou Express pour une mise en prodution effective.

A vous de jouer !

Tags: [Production, Wepack, React, ES6]

[precedent-article]: http://romaindso.github.io/react-webpack-starter/
[gist]: https://gist.github.com/romaindso/0495af3fdbcda48e5b9e
[lodash]: http://devdocs.io/lodash/
[ici1]: http://webpack.github.io/docs/cli.html#production-shortcut-p
[ici2]: https://github.com/webpack/docs/wiki/optimization#minimize
[localhost]: http://localhost:8080/
