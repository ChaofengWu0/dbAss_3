-- 不成立意为不合理  注释直接搜索“不成立”
create or replace function tin_check()
    returns trigger
as
$$
begin
    case (length(new.tin))
        -- 中国
        when 18 then begin
            case
                when (substring(new.tin, 9, 1) ~ '^[a-z]' and substring(new.tin, 10, 1) ~ '^[a-z]'
                    and substring(new.tin, 13, 1) ~ '^[a-z]' and substring(new.tin, 18, 1) ~ '^[a-z]')
                    then
                        if (isnumeric(substring(new.tin, 1, 8)) and isnumeric(substring(new.tin, 11, 2))
                            and isnumeric(substring(new.tin, 14, 4)))
                            -- 说明中国entity没问题
                        then
                            new.nation = 'cn';
                            new.format = '18';
                            new.type = 'entity';
                        else
                            -- 不成立
                            return null;
                        end if;

                -- 下面测中国人
                when (isnumeric(substring(new.tin, 1, 17)) and (substring(new.tin, 18, 1) = 'X' or isnumeric(new.tin)))
                    then
                        if
                            (isnumeric(new.tin) and cast(substring(new.tin, 18, 1) as integer) =
                                                    ((12 - (
                                                                       cast(substring(new.tin, 1, 1) as integer) * 7
                                                                   + cast(substring(new.tin, 2, 1) as integer) * 9
                                                                   + cast(substring(new.tin, 3, 1) as integer) * 10
                                                                   + cast(substring(new.tin, 4, 1) as integer) * 5
                                                                   + cast(substring(new.tin, 5, 1) as integer) * 8
                                                                   + cast(substring(new.tin, 6, 1) as integer) * 4
                                                                   + cast(substring(new.tin, 7, 1) as integer) * 2
                                                                   + cast(substring(new.tin, 8, 1) as integer) * 1
                                                                   + cast(substring(new.tin, 9, 1) as integer) * 6
                                                                   + cast(substring(new.tin, 10, 1) as integer) * 3
                                                                   + cast(substring(new.tin, 11, 1) as integer) * 7
                                                                   + cast(substring(new.tin, 12, 1) as integer) * 9
                                                                   + cast(substring(new.tin, 13, 1) as integer) * 10
                                                                   + cast(substring(new.tin, 14, 1) as integer) * 5
                                                                   + cast(substring(new.tin, 15, 1) as integer) * 8
                                                                   + cast(substring(new.tin, 16, 1) as integer) * 4
                                                                   + cast(substring(new.tin, 17, 1) as integer) * 2
                                                               ) % 11) % 11)
                                )
                        then
                            new.nation = 'cn';
                            new.type = 'person';
                            new.format = 'cn id';

                        else -- 不成立
                            return null;
                        end if;

                when (substring(new.tin, 1, 1) ~ '^[C,W,H,M,T]' and isnumeric(substring(new.tin, 2, 17))) then
                    new.nation = 'cn';
                    new.type = 'person';
                    case(substring(new.tin, 1, 1))
                        when ('C') then new.format = 'cn passport';
                        when ('W') then new.format = 'fn passport';
                        when ('H') then new.format = 'hk';
                        when ('M') then new.format = 'mo';
                        when ('T') then new.format = 'tw';
                        else return null;
                        end case;
                -- 不成立
                else return null;
                end case;
        end;

        -- bd entity
        when 11 then begin
            if (new.tin like 'RFC%' and (isnumeric(substring(new.tin, 4, 8))))
            then
                new.nation = 'bd';
                new.format = 'corporation';
                new.type = 'entity';
            else
                -- 不成立
                return null;
            end if;
        end;

        -- bd person
        when 9 then begin
            case
                when new.tin = '00-000000' then
                    -- 不成立
                    return null;
                when (isnumeric(substring(new.tin, 1, 2)) and (isnumeric(substring(new.tin, 4, 6))))
                    and (substring(new.tin, 3, 1) = '-')
                    then
                        begin
                            new.nation = 'bd';
                            new.type = 'person';
                            if (cast(substring(new.tin, 1, 2) as integer) > 50)
                            then
                                new.format = 'after 2000';
                            else
                                new.format = 'before 2000';
                            end if;
                        end;
                -- 不成立
                else return null;
                end case;
        end;

        -- rf entity
        when 10 then begin
            if (isnumeric(new.tin)) then
                if (exists(select code
                           from taxcodes
                           where code = substring(new.tin, 1, 4))
                    and (cast(substring(new.tin, 10, 1) as integer) = cast(substring(new.tin, 5, 5) as integer) % 10)
                    )
                then
                    new.nation = 'rf';
                    new.format = substring(new.tin, 1, 4);
                    new.type = 'entity';
                end if;

            else -- 不成立,不是数字
                return null;
            end if;
        end;

        -- rf person
        when 12 then begin
            if (isnumeric(new.tin)) then
                if (exists(select code
                           from taxcodes
                           where code = substring(new.tin, 1, 4))
                    and (cast(substring(new.tin, 11, 2) as integer) = cast(substring(new.tin, 5, 6) as integer) % 100))
                then
                    new.nation = 'rf';
                    new.format = substring(new.tin, 1, 4);
                    new.type = 'person';
                else
                    return null;
                end if;
            else -- 不成立,不是数字
                return null;
            end if;
        end;

        else return null;

        end case;
    return new;
end;
$$ language plpgsql;

-- drop trigger tin_trigger on tax_identification_number;


create trigger tin_trigger
    before insert
    on tax_identification_number
    for each row
execute procedure tin_check();
