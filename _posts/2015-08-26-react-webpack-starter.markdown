---
layout: post
title:  "React Webpack Starter"
date:   2015-08-26 23:43:01
description: React Webpack Starter
categories:
- blog
permalink: react-webpack-starter
---
On va voir comment mettre en place une stack de développement efficace pour travailler avec React grâce à Webpack. Histoire
d'exploiter toute la puissance de JavaScript, on va aussi utiliser les features ES6 (ou ES2015).

## Setup React
Pour gérer nos dépendances, on utilise `npm` et on démarre en installant React :

{% highlight console %}
$ npm install react --save
{% endhighlight %}

Avant d'aller plus loin, découvrons l'arborescence de notre projet React :

    - app
        - App.jsx
        - index.js
    - build
        - bundle.js **contenu géneré**
        - index.html

Passons rapidement en revue ces 3 fichiers :

**index.html**
{% highlight html %}
<html>
    <body>
        <div id="root"></div>
    </body>
    <script src="bundle.js"></script>
</html>
{% endhighlight %}

`index.html` c'est le fichier qui va inclure tout le reste. Lorsque l'on va lancer Webpack, ce dernier va compiler tout nos scripts JS dans le fichier `bundle.js`.

**index.js**
{% highlight javascript %}
import React from 'react';
import App from './App';

React.render(
    <App />,
    document.getElementById('root')
);
{% endhighlight %}

La repértoire `app` contient l'application React.
`index.js` est le point d'entrée de celle-ci. Il inclus évidemment React mais aussi le composant parent `App`. Le tout est rendu sur la div portant l'id **root** qui se trouve dans notre `index.html`.

**App.jsx**
{% highlight javascript %}
import React from 'react';

export default class App extends React.Component {

    render() {
        return (
            <h1>Hello world</h1>
        );
    }

}
{% endhighlight %}

Enfin, `App`, notre composant React parent qui inclura tous les autres. Oui oui, c'est bien du JavaScript mais à la sauce ES6. Notre composant est donc une classe qui étend la classe React.Component. Sa seule méthode, **render**, retourne pour l'instant qu'un simple *Hello World*. Enfin, pour simplifier l'écriture on utilise la syntaxe `jsx` pour écrire nos composants React.

## Wepack
Ok, le minimum vital pour React est présent, on enchaine maintenant avec [Webpack][webpack] :

{% highlight console %}
$ npm install webpack --save-dev
{% endhighlight %}

Difficile de décrire Webpack en 2 lignes. C'est avant tout un outil pour charger vos scripts JavaScript en tant que module, peut importe leur syntaxe (CommonJS, AMD). Grâce à différents loaders qu'on peut lui ajouter, Webpack est capable de consommer tout ce qui ressemble de près ou de loin à un fichier (CSS, images, json, font, etc). C'est à la fois un super task runner et un gestionnaire de module universel. Imbattable.

Wepack se paramètre grâce à un fichier de config, **le webpack.config.js** qui se place à la racine de votre projet.

**webpack.config.js**
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
    module: {
        loaders: []
    },
    resolve: {
        extensions: ['', '.js', '.jsx']
    }
};
{% endhighlight %}

Passons en revue les différentes lignes :

- entry : le point d'entrée de votre application
- output.path : le répertoire de sortie utilisé par Webpack
- output.filename : le fichier géneré qui contient tout nos scipts JS concaténés
- publicPath : la base url
- module.loaders : un tableau contenant les différents loaders
- resolve.extensions : Les extensions à résoudre par Webpack. On oublie pas de préciser l'extension jsx qu'on utlilise pour React.

On poursuit avec Webpack en lui ajoutant différents loaders.

#### Babel
Aujourd'hui la syntaxe ES6 n'est pas encore comprise par tous les navigateurs internet, on va donc utiliser [babeljs][babeljs] pour transpiler notre code en ES5. Cela va permettre d'utiliser les nouvelles [features][features] ES6 dès aujourd'hui !

{% highlight console %}
npm install babel-loader --save-dev
npm install babel-runtime --save
{% endhighlight %}

On complète notre **webpack.config.js** avec ce nouveau loader fraichement installé.
{% highlight javascript %}
...
loaders: [
    {
        test: /\.jsx?$/, // cible tous les fichiers en .jsx
        exclude: /node_modules/, // on exclut le répertoire node_modules
        loader: 'babel?optional[]=runtime' // et on précise quelques options
    }
]
...
{% endhighlight %}

#### CSS/SASS
Le chargement des CSS nécessite deux loaders : `css-loader` et `style-loader`. Le premier a pour but de résoudre les url() tandis que le second injecte directement les CSS dans une balise style au sein du body de la page. On ne va pas se priver d'écrire en SASS, on complète donc avec le `sass-loader` capable de gérer ce format et d'assurer la compilation. Par défaut, Webpack charge en inline toutes les feuilles de styles. Cela donne du code très difficile à débugger. Heuresement, grâce au plugin `ExtractTextPlugin` il est à la fois possible d'extraire nos CSS dans un seul fichier externe et d'activer les source maps pour s'y retrouver.

On installe les 3 loaders :
{% highlight console %}
npm install sass-loader css-loader style-loader --save-dev
npm install extract-text-webpack-plugin --save-dev
{% endhighlight %}

Et on met à jour notre config Webpack :
{% highlight javascript %}
...
devtool: 'source-map',
module: {
    loaders: [
        {
            test: /\.scss$/,
            loader: ExtractTextPlugin.extract(
                // activate source maps via loader query
                'css?sourceMap!' +
                'sass?sourceMap'
            )
        }
    ]
},
plugins: [
    // extract inline css into separate 'styles.css'
    new ExtractTextPlugin('styles.css')
]
...
{% endhighlight %}

#### Images et fonts
Pour le chargement des images et des fonts on va utiliser `url-loader`. On lui passe en paramètre une limite de poids en dessous de laquelle le loader charge le fichier en base 64 pour un gain de performance.
{% highlight console %}
npm install url-loader --save-dev
{% endhighlight %}

Ce qui donne :
{% highlight javascript %}
...
loaders: [
    {
        test: /\.(png|woff|woff2|eot|ttf|svg)$/,
        loader: 'url-loader?limit=100000'
    }
]
...
{% endhighlight %}

## Unleash the power !
#### Environnement de dev
On va enfin pouvoir s'atteler à faire tourner notre projet en local. Pour ce faire, Webpack fournit [webpack-dev-server][webpack-dev]. C'est un serveur Express (NodeJS) qui va nous permettre de servir tout nos fichiers
statiques. On est sur un mode complétement différent de Gulp ou Grunt qui eux utilise le système de fichiers via des tâches de copier/coller.

{% highlight console %}
$ npm install webpack --save-dev
{% endhighlight %}

Après l'avoir installé, il suffit d'ajouter un nouveau script au `package.json` :
{% highlight console %}
"scripts": {
  "start": "webpack-dev-server --content-base build/"
}
{% endhighlight %}

On lance enfin l'ensemble :
{% highlight console %}
$ npm start
{% endhighlight %}

Et on accède à [http://localhost:8080/][localhost]. <br> TADAAaaaa ! Un joli Hello World :)

#### Hot-reload
Webpack est capable de détecter les changements dans votre code et de relancer l'ensemble de l'application. C'est un puissant live-reload capable même de conserver l'état de vos composants React !

Pour l'activer il faut d'abord ajouter au fichier `index.html` la ligne suivante, juste avant le chargement du fichier `bundle.js` :
{% highlight html %}
...
<script src="http://localhost:8080/webpack-dev-server.js"></script>
...
{% endhighlight %}

On complète également avec une nouvelle entrée dans le webpack.config.js :
{% highlight javascript %}
...
entry: ['webpack/hot/dev-server', './app/index.js']
...
{% endhighlight %}

#### Eslint
Pour éviter les erreurs et les bugs, il est conseiller d'utliser un outil de qualité de code. Le nouveau venu c'est [ESLint][eslint] et il a de sérieux atouts. Compatible avec la syntaxe ES6 et le format JSX de React, il est aussi entièrement configurable.

Pour en profiter il faut rappatrier ESLint lui même, le plugin pour React et babel-eslint pour qu'il soit capable de lire le code transpilé par Babel. Enfin, il faut aussi le loader adéquat pour exécuter ESLint au lancement de Webpack.

{% highlight console %}
$ npm install babel-eslint eslint eslint-plugin-react eslint-loader --save-dev
{% endhighlight %}

ESLint se base sur un fichier de configuration pour définir les différents règles. Ce fichier c'est le `.eslintrc` et on le place en général à la racine de son projet (comme le webpack.config.js).
{% highlight javascript %}
{
    "ecmaFeatures": {
        "jsx": true,
        "modules": true
    },
    "parser": "babel-eslint",
    "env": {
        "browser": true,
        "node": true,
        "es6": true
    },
    "plugins": [
        "react"
    ],
    "rules": {
        "new-cap": 0,
        "strict": 0,
        "no-underscore-dangle": 0,
        "no-use-before-define": 0,
        "eol-last": 0,
        "quotes": [2, "single"],
        "react/jsx-boolean-value": 1,
        "react/jsx-quotes": 1,
        "react/jsx-no-undef": 1,
        "react/jsx-uses-react": 1,
        "react/jsx-uses-vars": 1,
        "react/no-did-mount-set-state": 1,
        "react/no-did-update-set-state": 1,
        "react/no-multi-comp": 1,
        "react/no-unknown-property": 1,
        "react/react-in-jsx-scope": 1,
        "react/self-closing-comp": 1
    }
}
{% endhighlight %}

Il faut maintenant exécuter ces règles côté Webpack. A la place d'un loader, on utilise un preloader, on s'assure ainsi que ESLint s'exécute avant toute autre opération (transpilation, compilation, etc etc).
**webpack.config.js**
{% highlight javascript %}
...
module: {
    preLoaders: [
        {
            test: /\.jsx?$/,
            loader: 'eslint-loader',
            exclude: /node_modules/
        }
    ]
}
...
{% endhighlight %}

## The end
Vous pouvez retrouver tout le code du projet sur mon github : <br> [https://github.com/romaindso/react-webpack-starter][github]

Et pour l'environnement de production ?
Ça sera l'objet d'un prochain post. A suivre...

Tags: [React, Wepack, ES6]

[webpack]: http://webpack.github.io/
[babeljs]: https://babeljs.io/
[features]: https://github.com/lukehoban/es6features
[webpack-dev]: http://webpack.github.io/docs/webpack-dev-server.html
[localhost]: http://localhost:8080/
[eslint]: http://eslint.org/
[github]: https://github.com/romaindso/react-webpack-starter
