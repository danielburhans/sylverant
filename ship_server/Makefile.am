#
#   This file is part of Sylverant PSO Server.
#
#   Copyright (C) 2009, 2010, 2011, 2012, 2013 Lawrence Sebald
#
#   This program is free software: you can redistribute it and/or modify
#   it under the terms of the GNU Affero General Public License version 3
#   as published by the Free Software Foundation.
#
#   This program is distributed in the hope that it will be useful,
#   but WITHOUT ANY WARRANTY; without even the implied warranty of
#   MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
#   GNU Affero General Public License for more details.
#
#   You should have received a copy of the GNU Affero General Public License
#   along with this program.  If not, see <http://www.gnu.org/licenses/>.

ACLOCAL_AMFLAGS = -I m4

LIBS += $(PTHREAD_LIBS) $(LIBICONV)
AM_CFLAGS = $(PTHREAD_CFLAGS)

bin_PROGRAMS = ship_server
ship_server_SOURCES = src/block.c src/block.h src/clients.c src/clients.h \
                      src/commands.c src/commands.h src/gm.c src/gm.h \
                      src/lobby.c src/lobby.h src/player.h src/ship.c \
                      src/ship.h src/ship_packets.c src/ship_packets.h \
                      src/ship_server.c src/shipgate.c src/shipgate.h \
                      src/utils.c src/utils.h src/subcmd.c src/subcmd.h \
                      src/list.c src/items.c src/items.h src/word_select.c \
                      src/word_select.h src/word_select-dc.h \
                      src/word_select-pc.h src/word_select-gc.h \
                      src/quests.c src/quests.h src/bans.c src/bans.h \
                      src/lookup3.c src/scripts.h src/scripts.c \
                      src/admin.h src/admin.c src/mapdata.h src/mapdata.c \
                      src/ptdata.h src/ptdata.c src/pmtdata.h src/pmtdata.c \
                      src/rtdata.h src/rtdata.c src/subcmd-dcnte.c

datarootdir = @datarootdir@
SUBDIRS = l10n
