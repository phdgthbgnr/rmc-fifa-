# rmc-fifa
### Réalisation d'un visuel à partager sur les RS d'une équipe de football

````js
//
// logic pour participation 1 
// du lundi au mercredi
(function(m,t,r,s,mbl,pnk){
    //"use strict";

    var token = t,
        refresh = r,
        semaine = s,
        ismobile = mbl,
        panik = pnk,
        selectionfifa = {},
        selections = {},
        curBddId = 0,
        datas = '',
        headerh = m.$dc('header').getBoundingClientRect().bottom;

    window.token = 0;
    window.refreshjson = 0;
    window.semaine = 0;
    window.panik = 0;
    
    // recupere la selection fifa de la semaine en cours (refresh) ---------------------------------------------
    if(refresh === 1){
        datas = 'token=' + token + '&action=semfifajoueur';
        m.promises.httpRequest('_services/endPoint.php', 'POST', datas, 9000).then(function(e){
            var resjson = JSON.parse(e);
            //sending = false;
        }).fail(function(error){

            if(error)console('erreur');
            sending = false;
        }).progress(function(progress){

        }).fin(function(){  // finally don't work on ie8 (ES5)
            sending = false;
        });
    }else{
        // charge seulement le json genere
        datas = '';
        m.promises.httpRequest('_json/selectionfifa-'+semaine+'.json', 'GET', datas, 9000).then(function(e){
            var resjson = JSON.parse(e);
            selectionfifa = resjson.data;
            start();
        }).fail(function(error){
            if(error) console('erreur');

        }).progress(function(progress){

        }).fin(function(){  // finally don't work on ie8 (ES5)

        });
    }
    // -----------------------------------------------------------------------------------------------

    var sending = false,
        curPoste = '',
        repjoueur = '_img/_joueurs/',
        repjoueurcnvs = '_img/_joueurscanvas/',
        divintro = m.$dc('intro'),
        divselection = m.$dc('selection'),
        divformulaire = m.$dc('formulaire'),
        divpartage = m.$dc('partages'),
        shareimg = m.$dc('shareimg'),
        scanvas = m.$dc('sharecanvas'),
        scontext = scanvas.getContext('2d'),
        nbJoueurs = 11, // <--------------------- NOMBRE DE JOUEURS IMPORTANT POUR GENERATION IMAGE DE PARTAGE
        indCanvas = 0, // <--------------------- NUMERO CURRENT PNG CHARGE DANS CANVAS
        qstr, // quesrystrinh shareing
        //compressed ='',
        isInSelection = function(id){
            var key = '';
            for (key in selections){
                if (selections[key]['joueurid'] == id) return true;
            }
            return false;
        },
        populatePopup = function(){
        // cles selection fifa
            var keysids = {'ag':'ailierg','g':'gardien','dc':'defenseurc','dg':'defenseurg','dd':'defenseurd','ad':'ailierd','bu':'buteur','mc':'milieuc','mdc':'milieudc'};
            var key = '';
            var idpop = '';
            for(key in selectionfifa){
                idpop = keysids[key];
                var htmlli = '';
                for(var k = 0; k < selectionfifa[key].length; k++){
                    var cur = selectionfifa[key][k];
                    var insel = isInSelection(cur['id']) == true ? ' insel' : '';
                    htmlli += '<li><a href="#' + key + '" id="joueur-' + cur['id'] + '" class="seljoueur' + insel + '"><img src="_img/nojoueur.png" alt="' + cur['prenom'] + ' ' + cur['nom'] + '" data-src-' + key + '="' + cur['image'] + '" /><p><span class="nom">' + cur['prenom'] + ' ' + cur['nom'] + '</span><span class="perf">' + cur['performance'] + '</span></p></a></li>'; //+'<!-- \n -->';
                }
                m.$dc(idpop).childNodes[1].childNodes[3].innerHTML = htmlli;
            }
            m.listenClass('seljoueur','click', setJoueurs, true);
        },
        setJoueurs = function(e,ct,t){
            var id = ct.getAttribute('id');
            //var id = e.currentTarget.id;
            var cnode = m.$dc(id);
            var sid = id.substring(7);
            selections[curPoste]['joueurid'] = sid;
            // for(var i = 0; i < e.currentTarget.parentNode.parentNode.childNodes.length; i++){
            for(var i = 0; i < e.currentTarget.parentNode.parentNode.childNodes.length; i++){
                var node = e.currentTarget.parentNode.parentNode.childNodes[i];
                var nid = node.firstChild.getAttribute('id');
                if (nid != id){
                    var snid = nid.substring(7);
                    if(m.hasAclass(nid,'insel') && !isInSelection(snid)) m.removeAclass(nid,'insel');
                }
            }
        
            if(!m.hasAclass(id, 'insel')) m.addAclass(id, 'insel');

            createCookie('rmcfifafut2017', selections, 30);

            testFini();

            populateSelections();

            var idp = cnode.parentNode.parentNode.parentNode.parentNode;
            m.addAclass(idp,'nodisplay');
        },
        testFini = function(){
            var fini = true;
            for(var k in selections){
                if(selections[k]['joueurid'] == '') fini = false;
            }

            if(fini) {
                m.removeAclass('continuer','nodisplay');
                m.addAclass('legende','nodisplay');
                window.scrollTo(0, document.body.scrollHeight);
            }else{
                m.removeAclass('legende','nodisplay');
                m.addAclass('continuer','nodisplay');
            }
        },
        closePop = function(e, ct, t){
            var id = e.target.parentNode.parentNode.parentNode.getAttribute('id');
            if(!m.hasAclass(id,'nodisplay')) m.addAclass(id,'nodisplay');
            return false;
        },
        clicJoueur = function(e, ct, t){
        //var id = ct.getAttribute('id');
            var id = e.currentTarget.id;
            for (var i = 0; i < ct.parentNode.parentNode.childNodes.length; i++){
                var elem = ct.parentNode.parentNode.childNodes[i];
                try{
                    var ha = elem.getElementsByTagName('a');
                    m.removeAclass(ha[0],'current');
                }catch(e){
                    // nothing
                }
            }

            m.addAclass(id, 'current');

            var poste = '';
            var idmodal = '';
            var datasrc ='';
            switch(id){
            case 'buteur1':
                poste   = 'bu';
                idmodal = 'buteur';
                datasrc = 'data-src-bu';
                break;
            case 'ailier1':
                poste   = 'ag';
                idmodal = 'ailierg';
                datasrc = 'data-src-ag';
                break;
            case 'ailier2':
                poste   = 'ad';
                idmodal = 'ailierd';
                datasrc = 'data-src-ad';
                break;
            case 'milieu1':
                poste   = 'mc1';
                idmodal = 'milieuc';
                datasrc = 'data-src-mc';
                break;
            case 'milieu2':
                poste   = 'mc2';
                idmodal = 'milieuc';
                datasrc = 'data-src-mc';
                break;
            case 'milieu3':
                poste   = 'mdc';
                idmodal = 'milieudc';
                datasrc = 'data-src-mdc';
                break;
            case 'defenseur1':
                poste   = 'dg';
                idmodal = 'defenseurg';
                datasrc = 'data-src-dg';
                break;
            case 'defenseur2':
                poste   = 'dc1';
                idmodal = 'defenseurc';
                datasrc = 'data-src-dc';
                break;
            case 'defenseur3':
                poste   = 'dc2';
                idmodal = 'defenseurc';
                datasrc = 'data-src-dc';
                break;
            case 'defenseur4':
                poste   = 'dd';
                idmodal = 'defenseurd';
                datasrc = 'data-src-dd';
                break;
            case 'gardien1':
                poste   = 'g';
                idmodal = 'gardien';
                datasrc = 'data-src-g';
                break;
            }

            // si joueur sélectionné dans milieu gauche on ne l'affiche pas dans milieu droit
            if( poste == 'mc1' ) listeDouble('milieuc','mc2');
            // si joueur sélectionné dans milieu droit on ne l'affiche pas dans milieu gauche
            if( poste == 'mc2' ) listeDouble('milieuc','mc1');
            // si joueur sélectionné dans defenseur central gauche on ne l'affiche pas dans defenseur central droit
            if( poste == 'dc1' ) listeDouble('defenseurc','dc2');
            // si joueur sélectionné dans defenseur central droit on ne l'affiche pas dans defenseur central gauche
            if( poste == 'dc2' ) listeDouble('defenseurc','dc1');
            curPoste = poste;
            if(m.hasAclass(idmodal, 'nodisplay')) m.removeAclass(idmodal, 'nodisplay');
            deferImages(function(){/*nothing*/}, datasrc, repjoueur);
            return false;
        },
        listeDouble = function(id1,id2){
            var curnode;
            var curid;
            var nodisplay;
            for(var i = 0; i < m.$dc(id1).childNodes[1].childNodes[3].childNodes.length; i++){
                curnode = m.$dc(id1).childNodes[1].childNodes[3].childNodes[i];
                curid = curnode.firstChild.getAttribute('id').substring(7);
                nodisplay = selections[id2]['joueurid'] == curid ? true : false;
                if(nodisplay){
                    m.addAclass(curnode.firstChild,'noselec');
                }else{
                    m.removeAclass(curnode.firstChild,'noselec');
                }
            }   
        },
        createCookie = function(name,value,days){
            var svalue = JSON.stringify(value);
            var expires = '';
            if (days) {
                var date = new Date();
                date.setTime(date.getTime() + (days*24*60*60*1000));
                expires = '; expires=' + date.toUTCString();
            }
            document.cookie = name + '=' + svalue + expires + '; path=/';
        },
        readCookie = function(name){
            var nameEQ = name + '=';
            var ca = document.cookie.split(';');
            for(var i=0;i < ca.length;i++) {
                var c = ca[i];
                while (c.charAt(0)==' ') c = c.substring(1,c.length);
                if (c.indexOf(nameEQ) == 0) return c.substring(nameEQ.length,c.length);
            }
            return null;
        },
        eraseCookie = function(name) {
            createCookie(name,'',-1);
        },
        clickStd = function(e, ct, t){
            var id = e.currentTarget.id;
            switch(id){

            case 'participer':
                m.removeAclass(divselection, 'nodisplay');
                m.addAclass(divintro, 'nodisplay');
                window.scrollTo(0, headerh);
                break;

            case 'continuer':
                m.addAclass(divselection, 'nodisplay');
                m.removeAclass(divformulaire, 'nodisplay');
                loadInCanvas();
                break;

            case 'valider':
                if(m.hasAclass('nom','erreur')) m.removeAclass('nom','erreur');
                if(m.hasAclass('prenom','erreur')) m.removeAclass('prenom','erreur');
                if(m.hasAclass('email','erreur')) m.removeAclass('email','erreur');
                if(m.hasAclass('mention','erreur')) m.removeAclass('mention','erreur');

                var err = new Array();

                var re = /[a-z0-9!#$%&'*+/=?^_`{|}~-]+(?:\.[a-z0-9!#$%&'*+/=?^_`{|}~-]+)*@(?:[a-z0-9](?:[a-z0-9-]*[a-z0-9])?\.)+(?:[A-Z]{2}|fr|com|org|net|gov|mil|biz|info|mobi|name|aero|jobs|museum)\b/;
                var email = m.$dc('email').value.toLowerCase();
                if (!re.test(email)) err.push('email');

                if (m.$dc('nom').value.trim() == '') err.push('nom');

                if (m.$dc('prenom').value.trim() == '') err.push('prenom');
                
                if(m.$dc('reglement').checked == false) err.push('mention');

                if (err.length == 0 && sending == false){
                    var frm = m.$dc('saveprofil');
                    var dts =  serialize(frm);
                    dts.push('selection=' + JSON.stringify(selections));
                    dts.push('semaine=' + semaine);
                    // datas.push('dataimg=' + compressed);
                    datas = dts.join('&');
                    sending = true;
                    m.addAclass('valider','nodisplay');
                    m.addAclass('retour','nodisplay');
                    m.removeAclass('patienter','nodisplay');
                    m.promises.httpRequest('_services/endPoint.php', 'POST', datas, 30000).then(function(e){
                        var resjson = JSON.parse(e);
                        if(resjson.error == 'ok') {
                            if (resjson.data.match('^([a-zA-Z0-9]){40}-([0-9]{6})$')){
                                curBddId = resjson.id;
                                m.$dc('imgshare').setAttribute('data-shrc', '_sharing/' + resjson.data + '.jpg');
                                m.$dc('download').setAttribute('href', '_sharing/_adwnld.php?token=' + token + '&shareimg='+ resjson.data);
                                m.$dc('facebook').setAttribute('href','#'+resjson.data);
                                m.$dc('twitter').setAttribute('href','#'+resjson.data);
                            }
                        }

                        deferImages(function(){
                            setTimeout(function(){
                                m.addAclass(divformulaire, 'nodisplay');
                                m.removeAclass(divpartage,'nodisplay');

                                m.removeAclass('valider','nodisplay');
                                m.removeAclass('retour','nodisplay');
                                m.addAclass('patienter','nodisplay');

                                sending = false;
                            },500);
                    
                        },'data-shrc','');

                        //sending = false;
                    }).fail(function(error){
                        if(error)console('erreur');
                        sending = false;
                    }).progress(function(progress){

                    }).fin(function(){  // finally don't work on ie8 (ES5)

                        //sending = false;
                    });
                }else{
                    for(var i=0;i<err.length;i++){
                        m.addAclass(err[i],'erreur');
                    }
                }
                break;

            case 'retour':
                m.addAclass('formulaire','nodisplay');
                m.removeAclass('selection','nodisplay');
                break;

            case 'download':
                break;

            case 'facebook':
                qstr = e.currentTarget.getAttribute('href').substring(1);
                var urlfic = 'http://trophee-fut-rmcsport.bfmtv.com/_sharing/'+qstr+'.jpg';
                /*
                FB.ui({
                    method: 'share_open_graph',
                    action_type: 'og.shares',
                    action_properties: JSON.stringify({
                        object: {
                            'og:url': 'http://trophee-fut-rmcsport.bfmtv.com/', //?imgsh='+qstr,
                            'og:title': "RMCSPORT FIFA UTIMATE TEAM",
                            'og:description': "Crée ton équipe FIFA ULTIMATE TEAM pendant 10 semaines",
                            'og:image': urlfic
                        }
                    })
                }, function(response){
    
                });
                */
                
                FB.ui({
                    method: 'feed',
                    //href: 'http://trophee-fut-rmcsport.bfmtv.com/?imgsh='+qstr,
                    name: 'RMCSPORT FIFA UTIMATE TEAM',
                    link: 'http://trophee-fut-rmcsport.bfmtv.com/?imgsh='+qstr,
                    picture: urlfic,
                    description: 'Crée ton équipe FIFA ULTIMATE TEAM et tente de gagner des cartes FUT !',
                    caption: 'RMCSPORT.BFMTV.COM'
                },function(d){
                    if(d !== undefined){
                        datas = 'token='+token+'&curid='+curBddId+'&action=sharefacebook';
                        m.promises.httpRequest('_services/endPoint.php', 'POST', datas, 30000).then(function(e){
                            //var resjson = JSON.parse(e);
                            //sending = false;
                        }).fail(function(error){
                            if(error)console('erreur');
                            sending = false;
                        }).progress(function(progress){
    
                        }).fin(function(){  // finally don't work on ie8 (ES5)
                            //sending = false;
                        });
                    }
                });
                
                break;

            case 'twitter':
                qstr = e.currentTarget.getAttribute('href').substring(1);
                var width  = 575,
                    height = 320,
                    left   = 300,
                    top    = 200,
                    text   = 'Crée%20ton%20équipe%20FIFA%20ULTIMATE%20TEAM%20et%20tente%20de%20gagner%20des%20cartes%20FUT%20!',
                    urlnflix ='http://trophee-fut-rmcsport.bfmtv.com/?imgsh='+qstr,
                    url    = 'https://twitter.com/intent/tweet/?text='+text+'&url='+urlnflix,
                    opts   = 'status=1' +
                ',width='  + width  +
                ',height=' + height +
                ',top='    + top    +
                ',left='   + left;
                window.open(url, 'twitter', opts);
                break;

            }
            return false;
        },
    
        rePopulateSelections = function(kk,ids){
            var kf = ''; // id selection fifa
            var idf = 'j' + kk; // id image à recharger
            switch(kk){
            case 'mc1':
            case 'mc2':
                kf = 'mc';
                break;
            case 'dc1':
            case 'dc2':
                kf = 'dc';
                break;
            default:
                kf = kk;
                break;
            }
            var oj= {};
            for(var i = 0; i < selectionfifa[kf].length; i++){
                oj = selectionfifa[kf][i];
                if(oj['id'] == ids){
                    m.$dc(idf).setAttribute('data-ssrc', oj['image']);
                    m.$dc(idf).setAttribute('alt', oj['prenom'] + ' ' + oj['nom']);
                }
            }
        
        },
        start = function(){
            m.listenClass('poste','click', clicJoueur, true);
            m.listenClass('closepopup','click', closePop, true);

            m.listenerAdd('valider','click', clickStd, true);
            m.listenerAdd('continuer','click', clickStd, true);
            m.listenerAdd('retour','click', clickStd, true);
        
            m.listenerAdd('facebook','click', clickStd, true);
            m.listenerAdd('twitter','click', clickStd, true);

            if(panik == 1){
                m.addAclass('participer','nodisplay');
            }else{
                m.removeAclass('participer','nodisplay');
                m.listenerAdd('participer','click', clickStd, true);
            }
        
            //m.listenerAdd('download','click', clickStd, true);

            var cook = readCookie('rmcfifafut2017');
            var err = false;
            if(cook !== null) {
                try {
                    selections = JSON.parse(cook);
                    if (selections['bu']['week']){
                        if(selections['bu']['week'] == semaine){
                            testFini();
                        }else {err = true;}
                    }else {err = true;}
                } catch(e){
                    err = true;
                }
            }else{
                err = true;
            }
            if(err){
            // x,y : position joueur sur image partage
                selections = {
                    'bu'    :{'joueurid':'','x':537,'y':1,  'x1':268,'y1':1,  'week':semaine},
                    'mc1'   :{'joueurid':'','x':404,'y':120,'x1':201,'y1':62, 'week':semaine},
                    'mc2'   :{'joueurid':'','x':676,'y':120,'x1':337,'y1':62, 'week':semaine},
                    'mdc'   :{'joueurid':'','x':540,'y':190,'x1':269,'y1':97, 'week':semaine},
                    'dg'    :{'joueurid':'','x':209,'y':248,'x1':105,'y1':127,'week':semaine},
                    'dd'    :{'joueurid':'','x':870,'y':248,'x1':434,'y1':127,'week':semaine},
                    'dc1'   :{'joueurid':'','x':385,'y':313,'x1':192,'y1':159,'week':semaine},
                    'dc2'   :{'joueurid':'','x':680,'y':313,'x1':340,'y1':159,'week':semaine},
                    'ag'    :{'joueurid':'','x':264,'y':44, 'x1':131,'y1':24, 'week':semaine},
                    'ad'    :{'joueurid':'','x':820,'y':44, 'x1':410,'y1':24, 'week':semaine},
                    'g'     :{'joueurid':'','x':536,'y':440,'x1':267,'y1':221,'week':semaine}
                };
            } 
        
            deferImages(function(){/*nothing*/},'data-src','');
            populateSelections();
            populatePopup();

        },
        populateSelections = function(){
            var k = '';
            for (k in selections){
                rePopulateSelections(k,selections[k]['joueurid']);
            }
            deferImages(function(){/*nothing*/},'data-ssrc',repjoueur);
        },
        deferImageCanvas = function(callback, ind, im, x, y, w, h, dx, dy, dw, dh){
            if (im.addEventListener != undefined){
                im.addEventListener('load',function(e){
                    callback(im, ind, x, y, w, h, dx, dy, dw, dh);
                });
            }else if (im.readyState){ // IE8
                im.onreadystatechange = function(){
                    if(im.readyState == 'loaded' || im.readyState == 'complete') {
                        callback(im, ind, x, y, w, h, dx, dy, dw, dh);
                    }
                };
            }
        },
        loadInCanvas = function(){

        // ajout bg
            indCanvas = 0;
            //scontext = scanvas.getContext('2d');
            scontext.save();
            scontext.setTransform(1, 0, 0, 1, 0, 0);
            scontext.clearRect(0, 0, 600, 315);
            scontext.restore();
            shareimg.src = '_img/blnkequipe.gif';
            var bimg = new Image();
            bimg.src = repjoueurcnvs + 'bg_share.jpg';
            deferImageCanvas(loadInCanvasPlayer, 0 , bimg, 0, 0, 600, 315, 0, 0, 600, 315);
        // loadInCanvasPlayer();
        },
        loadInCanvasPlayer = function(im, ind, x, y, w, h, dx, dy, dw, dh){
            //loadInCanvasPlayer = function(){
        // dessine l'image de fond 
            scontext.drawImage(im, x, y, w, h, dx, dy, dw, dh);
            //
            var imgs = new Array();
            for (var k in selections){
                var ks = k;
                if(k == 'mc1' || k == 'mc2') ks = 'mc';
                if(k == 'dc1' || k == 'dc2') ks = 'dc';
                for(var i = 0; i < selectionfifa[ks].length; i++){
                    if(selectionfifa[ks][i]['id'] == selections[k]['joueurid']){

                        imgs.push(new Image());
                        var l = imgs.length-1;
                        imgs[l].src = repjoueurcnvs + selectionfifa[ks][i]['image'] + '?req=' + Math.random();
                        var x = selections[k]['x1'];
                        var y = selections[k]['y1'];
                        // deferImageCanvas(drawInCanvas,l,imgs[l],0,0,124,184,x,y,124,184); // vignettes HD
                        deferImageCanvas(drawInCanvas,l,imgs[l],0,0,62,92,x,y,62,92);
                    }
                }

            }
        },
        drawInCanvas = function(im, ind, x, y, w, h, dx, dy, dw, dh){
        //var scontext = scanvas.getContext('2d');
        //scontext.save();
            scontext.drawImage(im, x, y, w, h, dx, dy, dw, dh);
            //scontext.restore();
        
            if (st) clearTimeout(st);
            if(indCanvas == nbJoueurs-1){
                var st = setTimeout(function(){
                    var data = scanvas.toDataURL();

                    shareimg.src = data;
                    m.$dc('dataimg').value = data; //Base64.btou(RawDeflate.inflate(Base64.utob(data)));
                    data = '';
                    // cdatas = '';
                },100);
            }
            indCanvas++;
        },
        deferImages = function(callback, dt, rep){
            var totImgDefer = 0;
            var imgDefer = document.getElementsByTagName('img');
            var nbDefer = [];
            // dt == data-srcslide
            for (var i=0; i<imgDefer.length; i++) {
                if (m.hasAclass(imgDefer[i],'nomobile') && ismobile == 1) continue;
                if(imgDefer[i].getAttribute(dt)){
                    nbDefer.push(imgDefer[i]);
                }
            }
            for (var i = 0; i<nbDefer.length; i++) {
                try{
                    nbDefer[i].setAttribute('src', rep+nbDefer[i].getAttribute(dt));
                }catch(err){
                    continue;
                }

                if (nbDefer[i].addEventListener != undefined){
                    nbDefer[i].addEventListener('load',function(e){
                        if(totImgDefer >= nbDefer.length-1) {
                            callback();
                        }
                        totImgDefer++;
                    });
                }else if (nbDefer[i].readyState){ // IE8
                    nbDefer[i].onreadystatechange = function(){
                        if(nbDefer[i].readyState == 'loaded' || nbDefer[i].readyState == 'complete') {
                            if(totImgDefer >= nbDefer.length-1) {
                                callback();
                            }
                            totImgDefer++;
                        }
                    };
                }
           
            }
        },
        handleError = function(e){
            switch(e.target.id){
            case 'imgshare':
                setTimeout(function(){
                    //m.dc('imgshare').setAttribute('src','');
                    //m.dc('imgshare').setAttribute('src','_img/back-share.png');
                    m.addAclass(divformulaire, 'nodisplay');
                    m.removeAclass(divpartage,'nodisplay');

                    m.removeAclass('valider','nodisplay');
                    m.removeAclass('retour','nodisplay');
                    m.addAclass('patienter','nodisplay');

                    sending = false;
                },500);
                break;
            }
            return false;
        },
        serialize = function(form) {
            if (!form || form.nodeName !== 'FORM') {
                return;
            }
            var i, j, q = [];
            for (i = form.elements.length - 1; i >= 0; i = i - 1) {
                if (form.elements[i].name === '') {
                    continue;
                }
                switch (form.elements[i].nodeName) {
                case 'INPUT':
                    switch (form.elements[i].type) {
                    case 'text':
                    case 'hidden':
                    case 'password':
                    case 'button':
                    case 'reset':
                    case 'submit':
                    case 'email':
                        q.push(form.elements[i].name + '=' + encodeURIComponent(form.elements[i].value));
                        break;
                    case 'checkbox':
                    case 'radio':
                        if (form.elements[i].checked) {
                            q.push(form.elements[i].name + '=' + encodeURIComponent(form.elements[i].value));
                        }
                        break;
                    case 'file':
                        break;
                    }
                    break;
                case 'TEXTAREA':
                    q.push(form.elements[i].name + '=' + encodeURIComponent(form.elements[i].value));
                    break;
                case 'SELECT':
                    switch (form.elements[i].type) {
                    case 'select-one':
                        q.push(form.elements[i].name + '=' + encodeURIComponent(form.elements[i].value));
                        break;
                    case 'select-multiple':
                        for (j = form.elements[i].options.length - 1; j >= 0; j = j - 1) {
                            if (form.elements[i].options[j].selected) {
                                q.push(form.elements[i].name + '=' + encodeURIComponent(form.elements[i].options[j].value));
                            }
                        }
                        break;
                    }
                    break;
                case 'BUTTON':
                    switch (form.elements[i].type) {
                    case 'reset':
                    case 'submit':
                    case 'button':
                        q.push(form.elements[i].name + '=' + encodeURIComponent(form.elements[i].value));
                        break;
                    }
                    break;
                }
            }
            return q;
        //return q.join("&");
        };   
    
    m.listenerAdd(window,'error',function(e){
        handleError(e);
    },true);

}(manageEvents, window.token, window.refreshjson, window.semaine, window.ismobil, window.panik));
````
