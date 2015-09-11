---
layout: post
title:  "Webpack pour la production"
date:   2015-09-10 21:35:01
description:
categories:
- blog
permalink: webpack-pour-la-production
---
Dans le [précédant article][precedant-article], on a vu comment mettre en place une stack de développement autour de React et Webpack. Parce que vous etes (surement) entrain de lancer le prochain Facebook, il va falloir créer la configuration adéquate pour packager l'appli en mode production. <br><br>Au programme : minification des assets, désactivation des sources-map et du débug et autres optimisations.

## Merge des configs Webpack
Pour s'aider, on va repartir du tuto précédant et réutiliser le **webpack.config.js** disponible sur ce [gist][gist]. Ce dernier correspond à une configuration de développement. L'idée c'est de conserver ce fichier pour le mode développement et d'en créer un second pour le mode production.

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
        path: path.join(/*__*/dirname, './build'),
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

#### Config de développement
Pour fusionner nos configs Webpack on va utiliser [lodash][lodash], une lib bien pratique qui permet d'utiliser de nombreuses méthodes utilitaires.

On l'installe via npm :
{% highlight console %}
npm install lodash --save
{% endhighlight %}

On oublie pas de faire un require de lodash et d'inclure également notre config de base.

Vient ensuite le code spécifique au mode dev. Rien de nouveau ici c'est le même code qu'il y avait dans le gist de départ mais déplacé dans le fichier adéquat. Ensuite, on utilise la méthode `merge` de lodash pour fusionner cette config avec celle de base. On passe à cette méthode une fonction pour préciser que les valeurs des propriétés de type array devront être concaténées (plutôt que remplacées). Enfin on exporte notre config.

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

#### package.json
Le **package.json** va être légerement modifié pour pointer vers les deux fichiers de configurations. L'option `--config` de Webpack permet justement d'indiquer un fichier différent du traditionnel **webpack.config.js**. Pour le mode de prodution on set la variable `NODE_ENV` à production pour permettre à nos modules et plugins d'agir en conséquence en réalisant diverses optimisations. Enfin, on rajoute l'option `-p` qui active entre autres la minification des assets.

{% highlight javascript %}
"scripts": {
  "dev": "webpack-dev-server --port 8080 --hot --inline --config webpack.dev.config.js",
  "deploy": "NODE_ENV=production webpack -p --config webpack.production.config.js"
}
{% endhighlight %}

Pour lancer notre config de dev on fera donc un `npm run dev` et pour notre config de prod un `npm run deploy`. Simple et efficace.


Tags: [Production, Wepack, React, ES6]

[precedant-article]: http://romaindso.github.io/react-webpack-starter/
[gist]: https://gist.github.com/romaindso/0495af3fdbcda48e5b9e
[lodash]: http://devdocs.io/lodash/


[features]: https://github.com/lukehoban/es6features
[webpack-dev]: http://webpack.github.io/docs/webpack-dev-server.html
[localhost]: http://localhost:8080/
[eslint]: http://eslint.org/
[rules-eslint]: http://eslint.org/docs/rules
