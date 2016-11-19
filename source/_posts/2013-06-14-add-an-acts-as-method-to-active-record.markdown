---
layout: post
title: "Add an 'acts_as' method to Active Record"
date: 2013-06-14 16:03
comments: true
categories:
---

Vediamo oggi come aggiungere i famosi metodi magici <em>acts_as</em> alle classi ActiveRecord.

La cosa da fare è create un modulo (magari dentro `lib`) con il nome del nostro acts_as:

    module Ciccio

      extend ActiveSupport::Concern

      included do
      end

      module ClassMethods  
        def acts_as_ciccio(add_power: false)
          # Do some stuffs
          if add_power
            include InstanceMethods2
          end
        end
      end

      module InstanceMethods
        def ciccio
          puts 'ciccio'
        end
      end

      module InstanceMethods2
        def ciccio2
          puts 'ciccio'
        end
      end

    end

    ActiveRecord::Base.send(:include, Ciccio) # optional

Caricando questo modulo all'avvio di Rails, tutte le vostre classi che derivano da `ActiveRecord::Base`
avranno i metodi di classe che sono definiti dentro il modulo `Ciccio::ClassMethods` e i metodi di istanza
che sono definiti dentro `Ciccio::InstanceMethods` o dentro `Ciccio`.
Nel nostro esempio inoltre possiamo aggiungere ad una classe i metodi di istanza dentro `Ciccio::InstanceMethods2`
solo se invochiamo il metodo di classe `acts_as_ciccio(add_power: true)` passando l'opzione `add_power` a true.
