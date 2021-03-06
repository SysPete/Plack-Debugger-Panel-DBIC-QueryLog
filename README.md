# NAME

Plack::Debugger::Panel::DBIC::QueryLog - DBIC query log panel for Plack::Debugger

# VERSION

Version 0.002

# SYNOPSIS

Adds a debug panel and querylog object for logging DBIx::Class queries.

Has support for [Catalyst](https://metacpan.org/pod/Catalyst) via a
[Catalyst::TraitFor::Model::DBIC::Schema::QueryLog](https://metacpan.org/pod/Catalyst::TraitFor::Model::DBIC::Schema::QueryLog) compatible trait,
[Catalyst::TraitFor::Model::DBIC::Schema::QueryLog::AdoptPlack](https://metacpan.org/pod/Catalyst::TraitFor::Model::DBIC::Schema::QueryLog::AdoptPlack).

       use Plack::Builder;
    
       use JSON;
    
       use Plack::Debugger;
       use Plack::Debugger::Storage;
    
       use Plack::App::Debugger;
    
       use Plack::Debugger::Panel::DBIC::QueryLog;
       use ... # other Panels

       use DBICx::Sugar qw/schema/;
       use MyApp;  # your PSGI app (Dancer2 perhaps)

       # create middleware wrapper
       my $mw = sub {
           my $app = shift;
           sub {
               my $env = shift;
               my $querylog =
               Plack::Middleware::DBIC::QueryLog->get_querylog_from_env($env);
               my $cloned_schema = schema->clone;
               $cloned_schema->storage->debug(1);
               $cloned_schema->storage->debugobj($querylog);
               my $res = $app->($env);
               return $res;
           };
       };

       # wrap your app
       my $app = $mw->( MyApp->to_app );
    
       my $debugger = Plack::Debugger->new(
           storage => Plack::Debugger::Storage->new(
               data_dir     => '/tmp/debugger_panel',
               serializer   => sub { encode_json( shift ) },
               deserializer => sub { decode_json( shift ) },
               filename_fmt => "%s.json",
           ),
           panels => [
               Plack::Debugger::Panel::DBIC::QueryLog->new,     
               # ... other Panels
           ]
       );
    
       my $debugger_app = Plack::App::Debugger->new( debugger => $debugger );
    
       builder {
           mount $debugger_app->base_url => $debugger_app->to_app;
       
           mount '/' => builder {
               enable $debugger_app->make_injector_middleware;
               enable $debugger->make_collector_middleware;
               $app;
           }
       };

# DESCRIPTION

This module provides a DBIC QueryLog panel for [Plack::Debugger](https://metacpan.org/pod/Plack::Debugger) with
query alaysis performed by [DBIx::Class::QueryLog::Analyzer](https://metacpan.org/pod/DBIx::Class::QueryLog::Analyzer) (by default).

For full details of how to setup [Catalyst](https://metacpan.org/pod/Catalyst) to use this panel and also for
a full background of the design of this module see
[https://metacpan.org/pod/Plack::Middleware::Debug::DBIC::QueryLog](https://metacpan.org/pod/Plack::Middleware::Debug::DBIC::QueryLog)
which this module steals heavily from.

# BUGS

Nowhere near enough docs and no tests so expect something to break somewhere.

This is currently 'works for me' quality.

Please report bugs via:

[https://github.com/SysPete/Plack-Debugger-Panel-DBIC-QueryLog/issues](https://github.com/SysPete/Plack-Debugger-Panel-DBIC-QueryLog/issues)

# SEE ALSO

[Plack::Debugger](https://metacpan.org/pod/Plack::Debugger), [Plack::Middleware::Debug::DBIC::QueryLog](https://metacpan.org/pod/Plack::Middleware::Debug::DBIC::QueryLog),
[Dancer2::Plugin::Debugger::Panel::DBIC::QueryLog](https://metacpan.org/pod/Dancer2::Plugin::Debugger::Panel::DBIC::QueryLog).

# ACKNOWLEDGEMENTS

John Napiorkowski, `<jjnapiork@cpan.org>` for [Plack::Middleware::Debug::DBIC::QueryLog](https://metacpan.org/pod/Plack::Middleware::Debug::DBIC::QueryLog) from which most of this module was stolen.

# AUTHOR

Peter Mottram (SysPete), `<peter at sysnix.com>`

# LICENSE AND COPYRIGHT

Copyright 2016 Peter Mottram (SysPete).

This program is free software; you can redistribute it and/or modify it
under the terms of either: the GNU General Public License as published
by the Free Software Foundation; or the Artistic License.

See http://dev.perl.org/licenses/ for more information.
