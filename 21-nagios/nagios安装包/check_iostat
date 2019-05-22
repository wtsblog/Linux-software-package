#!/usr/bin/perl -w
#
# check_iostats.pl - nagios plugin
#
# Copyright (C) 2007 Esben Bach <esben@ofn.dk>
#
# This program is free software; you can redistribute it and/or
# modify it under the terms of the GNU General Public License
# as published by the Free Software Foundation; either version 2
# of the License, or (at your option) any later version.
#
# This program is distributed in the hope that it will be useful,
# but WITHOUT ANY WARRANTY; without even the implied warranty of
# MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
# GNU General Public License for more details.
#
# You should have received a copy of the GNU General Public License
# along with this program; if not, write to the Free Software
# Foundation, Inc., 59 Temple Place - Suite 330, Boston, MA  02111-1307, USA.
#
# 15.02.2007 Version 1.0

use strict;
use Nagios::Plugin;
use Regexp::Common;

my $version = '0.2';
my %values;
my $iostat = '';
my $data = '';

my $np = Nagios::Plugin->new(
		usage => "Usage: %s -w|--warning=<warn> -c|--critical=<crit> [-i|--iostat=</path/to/iostat>]",
		version => $version,
		blurb => "Plugin that parses output from iostat to nagios",
);

sub handle_arguments()
{
	$np->add_arg(
			spec => 'w|warning=f',
			help => '-w, --warning=float - Percentage of I/O wait at which to enter status WARNING',
			default => 50, # Default value of warning is 50
			required => 1, # Mandatory argument!
		    );

	$np->add_arg(
			spec => 'c|critical=f',
			help => '-c, --critical=float - Percentage of I/O wait at which to enter status CRITICAL',
			default => 70, # Default value of critical is 70
			required => 1, # Mandatory argument!
		    );
	 $np->add_arg(
                        spec => 'i|iostat=s',
                        help => '-i, --iostat=path - full path to the iostat program',
                        default => undef, # No default value - uses that which is in the path of the nagios process
                        required => 0, # Optional argument!
                    );

	$np->set_thresholds(
			warning=> $np->opts->get('w'),
			critical=> $np->opts->get('c')
		);
}

sub get_data()
{
# Make some handling of iostat command here
	my $output = `$iostat -c`;

	# Return an unknown state if the iostat command was not executed successfully.
	unless($output) 
	{
		$np->nagios_die("Error executing iostat command");
	}

	#Pattern to match for avg-cpu followed by extra characters, treat all of the output as a single line ignoring \n
	$output =~ /avg-cpu.*?/gs;
	# Create map with variables
	for my $key (qw(user nice sys iowait idle))
	{
		#Yet another match this time continue after avg-cpu left off, and match the first real number and then stop
		#repeat this for every value in the map.
		if($output =~ /\G.*?($RE{num}{real})/gs)
		{
			$data .= sprintf("%s %s ", $key, $1);
		}
	}

	%values = split ' ', $data;
}

sub set_iostat()
{
	if ($np->opts->get('iostat'))
	{
		$iostat = $np->opts->get('iostat');
	}
	else
	{
		$iostat = 'iostat';
	}
}

sub perfdata()
{
#Add performance data variables
	$np->add_perfdata(
		label => 'iowait',
		value => $values{'iowait'},
		uom => '%',
		threshold => $np->threshold(),
	);

	$np->add_perfdata(
                label => 'idle',
                value => $values{'idle'},
                uom => '%',
		threshold => $np->threshold(),
        );

	$np->add_perfdata(
                label => 'user',
                value => $values{'user'},
                uom => '%',
		threshold => $np->threshold(),
        );

	$np->add_perfdata(
                label => 'nice',
                value => $values{'nice'},
                uom => '%',
		threshold => $np->threshold(),
        );
	$np->add_perfdata(
                label => 'sys',
                value => $values{'sys'},
                uom => '%',
		threshold => $np->threshold(),
        );
}

sub main()
{
# Main program
	handle_arguments();
	$np->getopts;
	set_iostat();
	get_data();
	
	perfdata();

	if ($values{'iowait'} < $np->opts->get('w'))
	{
		$np->nagios_exit( OK, "$data" );
	}
	elsif ($values{'iowait'} >= $np->opts->get('c'))
	{
		$np->nagios_exit( CRITICAL, "$data" );
	}
	elsif ($values{'iowait'} >= $np->opts->get('w'))
	{
		$np->nagios_exit( WARNING, "$data" );
	}
	else #No clue what value is there!
	{
		$np->nagios_exit( UNKNOWN, "Error in command output" );
	}
}

main();
